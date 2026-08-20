# Rust Microservice Project

A small microservice-based authentication system written in Rust.

The project demonstrates service-to-service communication using gRPC, asynchronous Rust with Tokio, authentication and session management, command-line interaction, automated health checks, and containerized deployment with Docker.

## Features

* gRPC communication using Tonic and Protocol Buffers
* User sign-up, sign-in, and sign-out
* PBKDF2 password hashing
* UUID-based user IDs and session tokens
* Command-line client built with Clap
* Automated health-check service
* Asynchronous execution with Tokio
* Dockerized microservices
* Docker Compose service orchestration
* Unit tests for authentication, users, and sessions

## Architecture

The project contains three main components:

### Authentication Service

The authentication service exposes a gRPC API on port `50051`.

It handles:

* Creating users
* Authenticating users
* Generating session tokens
* Signing users out
* Hashing and verifying passwords

User and session data are currently stored in memory using Rust `HashMap`s.

### Client

The client is a command-line application used to communicate with the authentication service.

It supports:

* `sign-up`
* `sign-in`
* `sign-out`

The client connects to the authentication service over gRPC.

### Health Check Service

The health-check service continuously tests the authentication service by:

1. Generating a random username and password
2. Creating a new account
3. Signing into the account
4. Signing out of the account
5. Repeating the process

This provides a simple service-to-service health check for the authentication API.

## Project Structure

```text
microservice-project/
├── .github/
│   └── workflows/
├── proto/
│   └── authentication.proto
├── src/
│   ├── auth-service/
│   │   ├── auth.rs
│   │   ├── main.rs
│   │   ├── sessions.rs
│   │   └── users.rs
│   ├── client/
│   │   └── main.rs
│   └── health-check-service/
│       └── main.rs
├── Cargo.toml
├── Cargo.lock
├── build.rs
├── Dockerfile-auth
├── Dockerfile-health
└── docker-compose.yaml
```

## Technologies

* Rust
* Tokio
* Tonic
* Protocol Buffers
* Prost
* PBKDF2
* UUID
* Clap
* Docker
* Docker Compose

## gRPC API

The API is defined in:

```text
proto/authentication.proto
```

The `Auth` service exposes three RPC operations:

```proto
service Auth {
    rpc SignUp (SignUpRequest) returns (SignUpResponse);
    rpc SignIn (SignInRequest) returns (SignInResponse);
    rpc SignOut (SignOutRequest) returns (SignOutResponse);
}
```

### Sign Up

Creates a new user.

```text
username
password
```

Returns a success or failure status.

### Sign In

Authenticates an existing user.

```text
username
password
```

A successful sign-in returns:

```text
userUuid
sessionToken
```

### Sign Out

Ends an active session using its session token.

## Running Locally

### Requirements

Install:

* Rust
* Cargo
* Protocol Buffers compiler (`protoc`)

Clone the repository:

```bash
git clone https://github.com/jedkutai/microservice-project.git
cd microservice-project
```

Build the project:

```bash
cargo build
```

## Running the Authentication Service

Start the gRPC authentication server:

```bash
cargo run --bin auth
```

The authentication service listens on:

```text
localhost:50051
```

## Using the Client

With the authentication service running, open another terminal.

### Create an account

```bash
cargo run --bin client -- sign-up --username jed --password password123
```

Short flags can also be used:

```bash
cargo run --bin client -- sign-up -u jed -p password123
```

### Sign in

```bash
cargo run --bin client -- sign-in --username jed --password password123
```

A successful login returns a user UUID and session token.

### Sign out

Use the session token returned by sign-in:

```bash
cargo run --bin client -- sign-out --session-token <SESSION_TOKEN>
```

## Running the Health Check

Start the authentication service first:

```bash
cargo run --bin auth
```

Then run the health-check service in another terminal:

```bash
cargo run --bin health-check
```

The health-check service will periodically send authentication requests and print the response status.

## Running with Docker

The authentication and health-check services can also be run using Docker Compose.

Build and start the services:

```bash
docker compose up --build
```

The Compose configuration starts:

```text
auth
health-check
```

The authentication service exposes port:

```text
50051
```

The health-check container connects to the authentication service using Docker's internal service networking.

Stop the services with:

```bash
docker compose down
```

## Authentication

Passwords are never stored directly.

When a user signs up:

1. A random salt is generated.
2. The password is hashed using PBKDF2.
3. The resulting password hash is stored with the user.

During sign-in, the supplied password is verified against the stored password hash.

Successful authentication generates a UUID-based session token.

## Current Limitations

This project is primarily a demonstration of Rust microservice architecture and currently has several limitations:

* User data is stored only in memory
* Session data is stored only in memory
* Data is lost when the authentication service restarts
* There is no external database
* Session expiration is not implemented
* The authentication service is not intended for production use

## What I Learned

This project was built to practice backend and distributed systems concepts in Rust, including:

* Designing gRPC APIs with Protocol Buffers
* Building asynchronous services with Tokio
* Implementing client/server communication with Tonic
* Structuring authentication logic in Rust
* Securely hashing and verifying passwords
* Managing application state safely across asynchronous requests
* Building command-line clients with Clap
* Containerizing Rust applications
* Connecting multiple services with Docker Compose
* Testing individual service components
