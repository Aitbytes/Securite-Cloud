---
title: 0-Breaking Kubernetes
---

Our mission here is to build a standardized cloud setup, dive into its inner workings, and spot the weak spots we might overlook. Sounds interesting, right? Well, let's get started!

As we navigate through these labs together, you'll see us using Infrastructure as Code (IaC) tools like Terraform and Ansible a lot. Why? Because they not only document every step automatically but also let us repeat our configurations without a hitch. This means you'll be able to follow along with ease and apply what you learn consistently.

### Why Not Go with GKE for Kubernetes Playtime?

You've probably heard about GKE, Google's managed Kubernetes service. It’s a great tool for managing many aspects of your Kubernetes clusters, like keeping the control plane running smoothly and handling upgrades and security updates automatically. Perfect for production, right? But what if you want to get your hands dirty?

If you’re looking to dive deep and tinker with Kubernetes itself, GKE isn't the best playground. Because Google manages so much of it for you, it limits how much you can experiment. Instead, using a setup like k3s on virtual machines (VMs) gives you the full reins. Imagine the freedom to poke around and fully understand Kubernetes architecture, exploring its different parts that a managed service usually hides away.

When you build Kubernetes on VMs with k3s, you’re not just following instructions—you’re crafting your own customizable environment. This is incredibly valuable for truly understanding Kubernetes and testing various scenarios hands-on.

**Having Fun with Kubernetes Vulnerabilities**: The fun part of these labs is discovering and exploiting vulnerabilities. By using k3s, you can tweak and even intentionally break things to see what happens. A managed service like GKE won’t let you do that—it has security and operational limits to keep everything smooth and safe, which isn't as fun when you’re trying to learn through experimentation.

Ready to dig in and explore? Let's see what we can uncover together! But before anything we must first [[Setting_environement |Set up the environnement]] . 
