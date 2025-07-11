---
layout: post
title: How Discode Performs Zero-Downtime Updates with Kamal Proxy
---

When I was building Discode, I needed a reliable way for customers to update their applications without downtime. Traditional deployment approaches often require stopping the old version before starting the new one, creating service interruptions that can be problematic for production systems.

Then I was inspired by the deployment tool Kamal. Behind the scenes, Kamal uses a lightweight reverse proxy called kamal-proxy to handle zero-downtime updates. This approach allows applications to be updated without any service interruption. This was exactly what I needed for Discode.

## The Problem with Traditional Updates

Some self-hosted applications suffer from the same update challenge. Stop your application, pull the new code, and start it again. This works fine for small applications or development environments, but in production, you often can't afford any downtime. Even if it's just a few seconds, that downtime can be noticeable to users and problematic for critical applications.

## Enter Kamal Proxy

Kamal's `kamal-proxy` is a lightweight reverse proxy that can route traffic between different versions of your application. Instead of the traditional stop-start approach, kamal-proxy allows you to:

1. Start the new version alongside the old one
2. Health check the new version
3. Switch traffic atomically
4. Remove the old version

This creates true zero-downtime deployments where users never experience service interruption.

## Implementing Kamal Proxy in Discode

For Discode, I needed to make this seamless update process available to customers who might not be familiar with container orchestration. The solution was to build this directly into the CLI tool that customers use to install and update their applications.

### The Installation Process

When customers install a Discode application, the CLI tool sets up the entire infrastructure:

```bash
discode install myapp
```

This command handles several steps:

1. Docker network creation.
2. Starts the kamal-proxy container.
3. Starts the application container.
4. "Deploying" the application to the kamal-proxy app for routing.

### The Update Process

The real magic with `kamal-proxy` happens during updates:

```bash
discode update myapp
```

Here's what happens under the hood:

1. The checks for a more recent release of the application.
2. Downloads the new Docker image
3. Starts the new updated application container
4. Checks the new container is responding correctly
5. Updates kamal-proxy routing to send traffic to the new version
6. Removes the old container after confirming the switch was successful

### Container Naming Strategy

Similar to how Kamal handles container names, Discode uses a specific naming convention for application containers. Each container is named using the format:

```
myapp-abc123def456
```

Each container includes the application slug and the release key. This allows multiple versions to coexist temporarily during updates, and makes it easy to identify which version is currently running.

This approach provides several advantages for Discode customers. Customers don't need to understand Docker networking or reverse proxy configuration. They just run `discode update myapp` and get a production-grade zero-downtime update.

## The Result

Now, when you use Discode to sell and distribute self-hosted Rails applications, you can provide your customers with a seamless update experience. They can deploy new versions of their applications without worrying about downtime or complex deployment procedures.

If you're building a self-hosted application and struggling with deployment complexity, I highly recommend exploring  kamal and kamal-proxy. Being able to dive into the details has been very insightful for Discode, and I hope it can help you too.
