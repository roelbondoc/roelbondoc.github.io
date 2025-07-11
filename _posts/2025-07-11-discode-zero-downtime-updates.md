---
layout: post
title: How Discode Performs Zero-Downtime Updates with Kamal Proxy
---

When I was building [Discode](https://rubyup.dev/discode), I needed a reliable way for customers to update their applications without downtime. Traditional deployment approaches often require stopping the old version before starting the new one, creating service interruptions that can be problematic for production systems.

Then I was inspired by the deployment tool [Kamal](https://kamal-deploy.org/). Behind the scenes, Kamal uses a lightweight reverse proxy called [kamal-proxy](https://github.com/basecamp/kamal-proxy) to handle zero-downtime updates. This approach allows applications to be updated without any service interruption. This was exactly what I needed for Discode.

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

### Installation Process

When customers install a Discode application, the CLI tool sets up the entire infrastructure:

```bash
discode install myapp
```

This command handles several steps:

1. Docker network creation.
2. Starts the kamal-proxy container.
3. Starts the application container.
4. "Deploying" the application to the kamal-proxy app for routing.

### Update Process

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

### `kamal-proxy` Commands

Using `kamal-proxy` took some time to understand, also required me to dive into Kamal to see how it was used. Here's what I learned:

This command will start the kamal-proxy container and binds the appropriate ports for HTTP and HTTPS traffic:

```bash
docker run --name kamal-proxy --network discode --publish 80:80 --publish 443:443 basecamp/kamal-proxy
```

Then we deploy the application to the kamal-proxy instance. This command tells kamal-proxy to route traffic for `myapp` to the specific myapp container:

```bash
docker exec kamal-proxy kamal-proxy deploy myapp --host myapp.hostname --target myapp-<release>:80
```

Now with any subsequent `kamal-proxy deploy`'s for `myapp`, it will route traffic to the latest version of the application container when that application is healthy.

### Container Naming Strategy

Similar to how Kamal handles container names, Discode uses a specific naming convention for application containers. Each container is named using the format:

```
myapp-abc123def456
```

Each container includes the application slug and the release key. This allows multiple versions to coexist temporarily during updates, and makes it easy to identify which version is currently running.

## The Result

Now, when you use Discode to sell and distribute self-hosted Rails applications, you can provide your customers with a seamless update experience. They can deploy new versions of their applications without worrying about downtime or complex deployment procedures.

This approach provides several advantages for Discode customers. Customers don't need to understand Docker networking or reverse proxy configuration. They just run `discode update myapp` and get a production-grade zero-downtime update.

If you're building a self-hosted application and struggling with deployment complexity, I highly recommend exploring  kamal and kamal-proxy. Being able to dive into the details has been very insightful for Discode, and I hope it can help you too.

## Try it Out

If you'd like to see how Discode works, I created a free web snapshotting service called [Webcap](https://rubyup.dev/webcap) that you can try out!
