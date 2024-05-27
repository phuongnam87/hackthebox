# Meow Write-up

\# BlueBrain

## Introduction

When first starting a penetration test or any security evaluation on a target, a primary step is known as `Enumeration`. This step consists of documenting the current state of the target to learn as much as possible about it.

Since you are now on the same Virtual Private Network (VPN) as the target, you can directly access it as any user would. If the target is a web server, running a public web page, you can navigate to its IP address to see what the page contains. If the target is a storage server, you can connect to it using the same IP address to explore the files and folders stored on it, provided that you have the necessary credentials. The question is, how do you find these services? You cannot manually search for them because it would take a long time.

Every server uses `ports` in order to serve data to other clients. The first steps in the Enumeration phase involve scanning these open ports to see the purpose of the target on the network and what potential vulnerabilities might appear from the services running on it. In order to quickly scan for ports, we can use a tool called `nmap` , which we will detail more in the Enumeration chapter of this write-up.

After finding the open ports on the target, we can manually access each of them using different tools to find out if we have access to their contents or not. Different services will use different tools or scripts to be accessed. These can be discovered and learned by a beginner penetration tester only with time and practice (and some diligent Googling). 90% of penetration testing consists of research done on the internet about the product you are testing. Since the technological ecosystem is continuously evolving, it is impossible to know everything about everything. The key is to know how to look for the information you need. The ability to research effectively is the skill you need to continuously adapt and evolve into your top quality.

The objective here is not speed but meticulousness. If a resource on the target is missed during the Enumeration phase of your test, you might lose a vital attack vector which would have potentially cut your worktime on the target in half or even less.

## Enumeration

After our VPN connection is successfully established, we can ping the target's IP address to see if our packets reach their destination. You can take the IP address of your current target from the Starting Point lab's page and paste it into your terminal after typing in the ping command as illustrated below.