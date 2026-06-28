---
title: "Docker: Why I Finally Stopped Fighting My Own Machine"
date: 2026-06-28 10:00:00 +0530
categories: [DevOps, Docker]
tags: [docker, containers, developer-tools, nodejs, react-native]
author: Pranav
---

Let me tell you something that happened at my office.

A colleague of mine a React developer came to me frustrated. He was working on two React projects and neither of them was running properly. We spent almost an hour debugging before we figured it out. His Node version was wrong. He had updated Node for one project, and it broke the other.

Sound familiar? Because the exact same thing happened to me.

If you've been building things for a while, you've had some version of this moment. Maybe it was Node, maybe Python, maybe Ruby. The point is: your machine was fighting you, and you didn't even know why.

That's exactly the kind of problem Docker came to solve.

---

## First, What Even Is a Container?

Before Docker, let me explain containers in the most common way I can.

Imagine you're cooking. You have one kitchen, one stove, one set of knives, one counter. Now imagine two recipes that need completely different setups. One needs the oven at 180°C, the other at 250°C. You can't do both at the same time in the same kitchen without things going wrong.

A **container** is like giving each recipe its own private kitchen. Same house, but completely isolated. Each kitchen has exactly what that recipe needs nothing more, nothing less.

In developer terms: a container packages your app along with everything it needs to run . the right Node version, the right dependencies, the right environment variables all isolated from everything else on your machine.

---

## Wait Wasn't This What Virtual Machines Did?

This is the question I had too. And it's a fair one.

Before containers, we had **Virtual Machines (VMs)**. The idea was similar isolate your environment. But the way it worked was very different.

A VM runs on top of something called a **hypervisor** (like VMware or VirtualBox). The hypervisor sits between your physical machine and the VMs. Each VM then gets its own full operating system — its own kernel, its own OS, its own everything.

It looks something like this:

```
Your Physical Machine
└── Hypervisor
    ├── VM 1 → Own Kernel + Own OS + App A
    ├── VM 2 → Own Kernel + Own OS + App B
    └── VM 3 → Own Kernel + Own OS + App C
```

This works. But it's *heavy*. Each VM carries an entire OS inside it. We're talking GBs just to spin up an isolated environment. Starting a VM takes minutes. Running 5 of them? Your laptop starts sounding like a jet engine.

**Containers solved this by doing one smart thing: sharing the kernel.**

Instead of each container having its own OS and kernel, containers all share the host machine's kernel. They're isolated from each other ,they can't see each other's files or processes but underneath, they're all using the same core of the operating system.

```
Your Physical Machine
└── Host OS + Kernel (shared)
    ├── Container 1 → App A (just the app + its dependencies)
    ├── Container 2 → App B (just the app + its dependencies)
    └── Container 3 → App C (just the app + its dependencies)
```

The result? Containers are tiny. They start in seconds, not minutes. You can run dozens of them on a normal laptop without breaking a sweat.

That's the real leap containers made over VMs not just isolation, but *lightweight* isolation.

---

## So What Is Docker?

Docker is the tool that lets you *create, run, and manage* those containers.

Think of Docker as the architect who builds those private kitchens. You tell Docker: "I need a kitchen with Node 16, this specific set of packages, and these environment variables." Docker builds it, hands you the keys, and you run your app inside it.

The app doesn't know or care what's on your actual machine. It only knows its own container.

---

## Why Did Docker Even Come to Exist?

There's a phrase every developer has said at least once in their career:

> **"It works on my machine."**

You build something. It runs perfectly on your laptop. You push it to a server, or hand it to a teammate, and suddenly — nothing works. Different OS, different versions, different environment.

This used to be a genuine nightmare. Teams spent days just making sure everyone's machines were set up the same way. Servers had to be carefully maintained so they matched developer machines. One wrong package update and everything broke.

Docker killed that problem. With Docker, you don't share code and *hope* the other person's machine is set up right. You share the entire environment —> code + dependencies + OS layer + config, all in one box.

Now when someone says "it works on my machine," they can just ship their machine.

---

## But Wait  Does Docker Work for Everything?

Honestly? No. And this is something I wish someone had told me earlier.

**Docker works really well for:**
- Backend services (Node.js APIs, Python apps, databases)
- Web frontends (React, Next.js running in a browser)
- DevOps, servers, cloud deployments

**Docker does NOT work well for mobile development.**

If you're building React Native apps, you generally don't write a Dockerfile. You don't need to. Mobile development needs:
- **Android Studio** with an emulator (or a real Android device)
- **Xcode** (on macOS) for iOS
- **Simulators** that talk directly to your machine's hardware and GPU

Containers are isolated from all of that. There's no screen, no simulator, no way to practically run an Android emulator inside a container. The container has no idea your phone exists.

So in the mobile world, developers just don't use Docker for the app itself. That's where **NVM** comes in for the Node version problem. it's the right tool there.

But the moment you step into **web or backend** — that's where Docker is everywhere. React web apps, Node APIs, Python services, databases — teams actively write Dockerfiles and use Docker Compose for all of this. It's become a standard part of the workflow.

So the rule of thumb:
- **Mobile apps (React Native, Flutter)** → no Dockerfile, use NVMFVM for version management
- **Web apps (React, Next.js)** → Docker is commonly used
- **Backend (APIs, databases, services)** → Docker is almost always there

---

## The Node Version Problem (You've Felt This)

Okay, back to my embarrassing afternoon.

I was working on two React projects at the same time:
- **Project A** — an older React app that needs Node 16
- **Project B** — a newer React app that needs Node 18

On a normal machine, you can only have one "active" Node version at a time. So I'd switch to Project B, update Node, work fine — then switch back to Project A and it would break.

**The NVM way**: [NVM (Node Version Manager)](https://github.com/nvm-sh/nvm) solves this partially. You can do `nvm use 16` or `nvm use 18` and switch between versions manually. It's honestly great and I still use it.

But here's what NVM *doesn't* solve:
- What if your teammate doesn't have NVM set up?
- What if you're deploying to a server?
- What if Project A needs specific environment variables or a specific npm registry?
- What if you're running 5 projects and keep forgetting to switch versions?

This is where Docker shines. Each project gets its own container with the exact Node version baked in. You don't switch anything. You just start the container and work.

```bash
# Without Docker - you have to remember to switch
nvm use 16
npm start

# With Docker - the container already knows what version it needs
docker compose up
```

No thinking. No switching. No "why isn't this working."

---

## So How Does Docker Actually Isolate Things?

I remember asking this exact question when I first learned that containers share the kernel. My immediate thought was okay but if everything is sharing the same kernel, what stops one container from seeing another container's files? Or its processes?

Turns out Linux already had the answer, way before Docker even existed. It's called **namespaces**.

When Docker starts a container, it tells the Linux kernel . hey, wrap this process in its own namespace. And the moment that happens, the container goes into its own little world. It can't see your processes. It can't see your files. It doesn't even know other containers exist. From inside the container, it genuinely feels like it's the only thing running on the machine.

I think of it like noise-cancelling headphones. You're still in the same room as everyone else, same building, same air. But you can't hear anything outside. That's a namespace.

And there's one more piece **cgroups**. Short for control groups. This one is about limits. Namespaces handle *what* a container can see, but cgroups handle *how much* it can take. How much CPU, how much memory, how much disk. Without this, one container could go rogue and eat up all your RAM and starve every other container running beside it.

---

## Where I Am Right Now

I'm still learning Docker. I've gone from "what even is this" to actually understanding why it exists and what problem it solves and honestly, that shift in understanding made everything click.

The commands are still a bit intimidating sometimes. A `Dockerfile` can look scary the first time. But once you realize it's just a recipe "start with this base image, install these things, copy my code here, run this command" it starts making sense.

If you're in that same "I get the concept but not sure how to use it" phase, that's exactly where I was. And we'll figure it out together.

More Docker posts coming. Next up: writing an actual Dockerfile from scratch.
