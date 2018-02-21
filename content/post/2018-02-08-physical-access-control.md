---
title: Physical Access Control
date: 2018-02-08
---

  Access control has its own meaning in software development so this might confuse some people. In this case the system was used to control the access to certain places: a building, an office or maybe a specific room inside the office (like a datacenter).  
<!--more-->  

  This system provided a way to control visitors in a building reception, to define times and doors where employee badges would have access and also allow security team to monitor who was entering or leaving the place.

### Pros

- Usability: many features created after direct observation of users.
- Support of multiple databases (Access, Sql Server, Oracle).

### Cons

- No automated tests
- No version control
- Spaghetti code

## Architecture & Evolution

### First generation - One app for everything

<Diagram>

  The system started as a simple Windows desktop application for VB4. This application was responsible to register new users (eg employees, contractors, visitors), who were allowed in each access point (door, turnstiles, gate). The access points were controlled by dedicated controllers which communicated with the main application sending events processed (entrances, exits) and receiving updates in permissions on that point.

  The first problem with this approach is that the register of users and communication with devices were very different problems to solve in the same application. If we have two people working in the reception, each instance of the application would try to communicate with all controllers in the area. Also, which instance would be responsible for receiving events? Only one should have this role, to avoid registering events twice.

### Second Generation 

<Diagram>

  The solution for the problems in the first generation was to extract the integration with controllers from the main application and create another one dedicated to it. This new application was supposed to run without any direct user action, just communicating with controllers and processing their events. 

  Now, there was still the need to update the controllers when new permissions were issued. Since the direct communication didn't exist anymore, the main module should tell the integration module what changed. For this an event table was created in the database, that would work as a notification system to the integration module. The main app would insert in this tables users that had their permissions changed and the integration module would query these tables and make sure that the controllers received the new permissions.

  This worked well for some time but another problem happened over time. These controllers we used for controlling the physical equipment (door, turnstile) had many vendors, each one with its own hardware and communication protocol. You would usually receive a SDK of each vendor to integrate your application with it. When the integration module was talking to 5 different vendors, the code just became too hard to maintain. Notice here that the lack of isolation and layers was the main cause of this issue. A proper use of interfaces to encapsulate specific logic to vendors would have made the trick but back then we didn't know these concepts.

### Third (and final) generation 

<Diagram>

  The solution for that was to split the application again, creating one module for each vendor and this time, also creating a single module to visualize the events happening. This last one was commonly used by the security team of a building, for instance. You might even say that we were moving towards a microservice architecture by then! 

  That was the last architectural change that happened in the system while I was there. It is interesting to notice that even with all the bad practices and lack of patterns the split of the system in small modules made the maintenance way easier. Even a badly written system is not a big deal, if you can fit most of it at your head at once.

## Lessons learned

We didn't have a version control system, so working in teams was a mess. The code was in a shared folder in the network and everyone needed to be very careful to not overwrite someone else's code. Our first try to resolve this was to use Source Safe and this managed to make things even worse!

We didn't have any constraints in our database (maybe some primary keys) and no transaction control in our code. That caused a lot of bugs.
Database migration was a real nightmare. There was no formal process or tool to control the database evolution. Imagine that you have 50 customers with different versions of your system and different versions of the database. We even built a tool to automatically diff different instances of the database, so we could manually apply the migrations script.
	
You don't have to start your system with a microservice architecture but it helps a lot with maintenance splitting the system in smaller parts when they start doing too much.
