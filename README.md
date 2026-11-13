<picture><source media="(prefers-color-scheme: dark)" srcset="https://nextwork.ai/static/nextwork-logo-paper.svg" /><img src="https://nextwork.ai/static/nextwork-logo-leather.svg" alt="NextWork" width="300" /></picture>

# URL Shortener System Design

**Project Link:** [View Project](https://nextwork.ai/projects/02a35a5b-1d8b-43ee-9982-6ceef44145e5)

---

## Project Overview

### What we're building and why

In this project, I'm building a URL shortener so that I can converts long URLs into short codes and redirects users, powered by a Python Flask API, a PostgreSQL database, and a Redis cache running together on the machine.

## Setting Up the Environment

### Environment setup goals

In this step, I'm setting up my environment,i.e, a terminal for running commands, Docker for running each service as a container, and a code editor for working with your project files so that I can build a URL shortener with multiple services.

![](https://nextwork.ai/secure_beige_smart_jujube/uploads/img_1794544966620_184e8656.png)

![](https://nextwork.ai/secure_beige_smart_jujube/uploads/img_1794544952620_4242de3a.png)

### The three core services

The three services are API server, database and cache.  The first handles receiving incoming requests (like "shorten this URL" or "redirect me") and route them to the right place. The second stores the mapping between short codes and their original long URLs so they persist even after restarts. The third provides frequently accessed URL mappings in memory so popular links redirect instantly without hitting the database every time.

## Designing the System Architecture

### Architecture design goals

In this step, I'm creating the infrastructure files so that I can define how the services are built, connected and started. So in short I will be defining the three-services architecture in a Docker Compose file, create the container build recipe, Python dependencies, database schema and understand why PostgreSQL is the right storage chocie for this system.

![Designing the System Architecture](https://nextwork.ai/secure_beige_smart_jujube/uploads/02a35a5b-1d8b-43ee-9982-6ceef44145e5_pnungqem)

### Services defined in docker-compose.yml

The three services are web, db and redis. Each one handles different services that works together:\
1\. web is the Python API server. It builds from the ./app folder and exposes port 5000 on the machine.\
2\. db is a PostgreSQL database that stores URL mappings permanently. The healthcheck uses pg\_isready -U postgres to confirm it's accepting connections.\
3\. redis is a Redis cache for fast lookups. Its healthcheck uses redis-cli ping to confirm it's ready.\
The depends\_on with condition: service\_healthy ensures the database and cache are fully ready before the API server starts.

![Designing the System Architecture](https://nextwork.ai/secure_beige_smart_jujube/uploads/02a35a5b-1d8b-43ee-9982-6ceef44145e5_ymp8agg2)

### Infrastructure file deep dive

I chose Dockerfile. Its role is a recipe for building a container image. This one starts from a slim Python 3.12 base image, installs the packages from requirements.txt, copies your application code into the image, and runs app.py when the container starts. In plain english, its role is a recipe card for building a mini-computer. It says: start with a fresh computer that already has Python, install the shopping list, copy your code in, and run it. Every time Docker builds your app, it follows this recipe from the top.

## Building the URL Shortener Logic

### Application logic goals

In this step, I'm writing the brain of the system so that I can create app.py, the Python file that handles both jobss of a URL shortener: coverting long URLs into short codes, and redirecting short codes back to the original URL.

### The URL shortening flow

When a long URL hits /shorten, the app first stores it in the PostgreSQL, then it gets the database ID, encodes it, caches it, and finally returns the short URL. \
The full flow looks like this:

Receive long URL → store in database → get a unique ID back → encode that ID into a short code → cache it → return the short URL.

![Building the URL Shortener Logic](https://nextwork.ai/secure_beige_smart_jujube/uploads/02a35a5b-1d8b-43ee-9982-6ceef44145e5_seusouy2)

### Handling redirect lookups

When someone visits a short URL, the app first checks which original URL it points to and send them there instantly. Speed matters here because redirects happen far more often than shortening. If that misses, it falls back to PostgreSQL (slower).\
redirect\_url() handles GET requests. It first checks Redis (fast), falls back to PostgreSQL (slower) , caches the result for next time, and issues a 302 redirect.

The cache-first pattern is a core system design technique. Checking the fast cache first and only bothering the slower database on a miss keeps redirects snappy under load.

## Launching and Verifying the System

### Launch goals

In this step, I'm starting all three containers with a single Docker Compose command so that I can confirm the Flask API responds to a health check and verify PostgreSQL and Redis are connected through their logs.

![Launching and Verifying the System](https://nextwork.ai/secure_beige_smart_jujube/uploads/02a35a5b-1d8b-43ee-9982-6ceef44145e5_3w4aoken)

### Health endpoint response

When I ran curl http://localhost:5000/health, I saw {"status":"healthy"} in the content.

## Testing and Observing the System

### Testing goals

In this step, I'm creating short URLs via the API and observe Base62-encoded short codes, testing URL redirection so that I can observe HTTP status codes and observe the caching layer in real time.

### Redis cache flush behavior

After flushing Redis, the app had to query PostgreSQL instead of Redis. In other words, app queried PostgreSQL after the cache miss, served the redirect, and stored the result back in Redis for next time. That's the cache-aside pattern in action. Cache-aside means the app is responsible for managing what goes into the cache. When someone visits a short URL, the app checks Redis first. If Redis doesn't have the answer, the app queries PostgreSQL and saves the result into Redis so the next request is faster.

The alternative is "write-through," where the app writes to both the cache and database at the same time. Cache-aside is simpler to build but the cache can get out of date. Write-through keeps things in sync but slows down every write.

## Implementing Rate Limiting with Redis

![Implementing Rate Limiting with Redis](https://nextwork.ai/secure_beige_smart_jujube/uploads/02a35a5b-1d8b-43ee-9982-6ceef44145e5_i71om2um)

### Why Redis excels at rate limiting

In this project extension, Redis is better for rate limiting because Redis INCR is atomic, meaning it is thread-safe even under concurrent requests. Redis also supports key expiration (TTL), so you can increment a counter per IP and auto-reset it every 60 seconds without any cleanup jobs.\
This is how Cloudflare, Stripe, and GitHub implement rate limiting.

## Reflections and Key Takeaways

### Tools and concepts learned

The key tools I used include Python API, PostgreSQL database, and Redis cache. Key concepts I learnt include:\
1\. Build a multi-service distributed system with a Python API, PostgreSQL database, and Redis cache that shortens URLs and redirects users.

2\. Apply the cache-aside pattern and explain 301 vs 302 redirect trade-offs the way interviewers expect.

3\. Use Docker Compose to orchestrate containers with health checks, networking, and startup dependencies.

4\. Secret Mission: Implemented Redis-based rate limiting with atomic counters and TTL expiration to prevent API abuse.

### Time and challenges

This project took me approximately an hour to complete. The most challenging part was understanding how three services work and how they can be build into multi-service distributed system using Docker.

### Final thoughts

I did this project today to learn how to build a multi-service distributed system with a Python API, PostgreSQL database, and Redis cache that shortens URLs and redirects users.
