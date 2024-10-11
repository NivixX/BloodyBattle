<p>
  <img src="https://github.com/NivixX/BloodyBattle/assets/5229597/5c0ef793-88d2-4e54-8814-8ef7a3bcc1aa" width="249" height="100" />
  <img src="https://github.com/user-attachments/assets/73092711-768a-4a0a-a3e7-9808249f0314" alt="GIF Presentation" width="400" height="225" />
</p>
<br />

## What is BloodyBattle
**BloodyBattle is a Java minecraft server I founded and developed for over 11 years (from 2012 to 2023). Known as the leader in the French "Pvp/Faction" community, and was part of my server network that has many games (such as HungerGames, Bedwars, ...)**

### Some numbers
- **1.500.000** total registered users
	- **1M** from mini-games network including HungerGames
	- **500k** exclusively from BloodyBattle
- Record of **2000 concurrent online players** in HungerGames using **24 parallel server instances**
- Record of **500 concurrent online players** in BloodyBattle **in one single instance** with over 110 loaded plugins, most of them are highly optimised using async [NMS protocol hack](https://wiki.vg/Protocol "NMS protocol hack") (through netty threads)

### What is the content of this repository?

This repository is a **collection of plugins**, each accompanied by a brief description and, where relevant, **technical explanations with diagrams**. I believe that developing a large-scale Minecraft server is a fantastic exercise in mastering IT engineering, as it presents a wide range of **challenges**, including data structures, real-time constraints (balancing asynchronous optimization with state consistency), netcode, DevOps, and more.

Like many, I began my journey into the tech industry through game server software development. If you're also passionate about game servers, I hope you'll find something valuable here to enhance your knowledge and skills.

At this time, the **full source code of BloodyBattle** won't be made public, as I want to avoid the potential for other servers to simply copy my game design and hard work development. Additionally, I may decide to launch a new, modern version of **BloodyBattle v2** in the future. However, I will share parts of the code depending on the game's state and my future plans. For those interested, my **NDatabase** plugin—used to manage BloodyBattle's database—is fully open-source and available [here](https://github.com/NivixX/NDatabase).

Despite the current limited source code, I hope the **technical explanations and insights** provided in this repository will be valuable to developers and game server enthusiasts alike, especially if you're interested in achieving **optimal performance for high-scale servers with a large number of players**.


-----

#### BloodyCustomItems
A plugin that add new complex items such as **catapults** used to raid base, treasure generator, special tools, etc. Most of these items are highly performant, because it run async using NMS.

<img src="https://github.com/user-attachments/assets/ae28ffeb-eeb3-490e-8deb-e552d34eba76" width="533" height="300" />
<br />

> Showcase of a catapult, it can throw tnt with customisable height, power, rotation. The ennemy team can also destroy the catapult by hitting it during a siege.

-----

#### EpicAnalytics
Advanced Real Time Analytics system plugin using **Grafana** and **InfluxDB**

<img src="https://github.com/user-attachments/assets/cbaf4e10-22dd-4bdc-8ec5-b2eecff27279" width="800" height="350" />
<br />

> Example of server performance monitoring containing player count, ram/cpu usage, TPS (how much the server tick per seconds), and tick duration (how much ms it take to apply the main thread logic tick)

<br />

<img src="https://github.com/user-attachments/assets/e7c4ff85-c445-450c-82c2-7b0609dbe54e" width="556" height="350" />
<br />

> Example of a real time leaderboard for every top teams on the server

<br />

<img src="https://github.com/user-attachments/assets/70369177-6266-40b5-b020-39d21f4d4275" width="758" height="350" />
<br />

> Example of a survey tracking system, poll are sent to players in-game using the plugin BloodySurvey

The API is very easy to use, for the above example with the survey, here is how I do to add a new `InfluxDB Point` that represent the survey response of a player
```java
EpicAnalytics.getController().writeAsync(TrackedEntry.measurement("survey-v14")
	.tag("survey-id", surveyId)
	.tag("survey-response-id", responseId)
	.addField("survey-response", response.get().getAnswerValue())
	.build());

```
-----

#### BloodyPignata
A looting event where players need to hit a pinata and receive lot, the visual and mechanics of the pinata has been done fully async using NMS.
The pinata has dynamic max health depending on the number of players in the event, every player receive multiple clickable loot boxes with reward proportional to their contribution to the event.

<img src="https://github.com/user-attachments/assets/6ca1b327-51c9-4f39-bfc2-3168f825acac" width="533" height="300" />


-----

### EpicAPI - Minecraft Plugin SDK

**EpicAPI** is an internal SDK I built and use across most of my plugins. The primary feature of this SDK is a rich API for manipulating **NMS (net.minecraft.server) via protocol hack with Netty**, enabling advanced functionality like dynamically instantiating entities with customized behavior for each individual player, all in a fully asynchronous manner. 

This is achieved through the **`Structure` API**, which I will explain and provide examples of later in the documentation. In addition to NMS manipulation, **EpicAPI** addresses other common plugin needs, such as:

- Building in-game **GUI systems** (inventory-based, anvil, text input)
- **Particle effects** with geometry utilities
- A versatile **scheduling system**
- etc

First, let's explore one of the **most significant performance gains** you can achieve using **NMS**. To start, we need to understand the basics of how a Minecraft server operates and manages the game context.

![epicApi-Page-1](https://github.com/user-attachments/assets/24ee72a2-7f6c-4a27-811d-6deaf64198df)


As shown in the diagram above, the server ticks every **50ms** at minimum, which translates to a maximum of **20 ticks per second (TPS)**. Within each tick, the server has to synchronize and process a wide range of tasks—updating entities, tiles, AI, movements, chunks, and broadcasting changes to all connected players. On large servers with many players, loaded chunks, and entities, this processing can quickly exceed the 50ms threshold, resulting in noticeable **lag** for all players.

The key to mitigating this is **reducing the workload** on the main thread. The diagram below shows an approach to solve this challenge.

<img src="https://github.com/user-attachments/assets/f5ca3628-a8c1-4901-99e8-e485b705c283" height="700" />

The idea behind **EpicAPI** is to listen directly to the **Netty thread** in the network layer and create a parallel **asynchronous context** for handling interactions between players and fake entities. These fake entities are sent to players by constructing and sending Minecraft network packets manually through the Netty thread. By doing this, you can introduce features for your players without relying on the main server thread.

This method works for everything—chunks, blocks, items, movements, etc. It also enables more **advanced features** that are only possible with **NMS**. For example, you can spawn armor stand entities and customize the packets sent to each player, displaying different text values for different players or showing entities only to certain players.

Additionally, you may want to modify the main game state by giving items to players, modifying the world, etc. In that case, from the asynchronous context, you can schedule a task that will modify the game state and be executed on the main thread, ensuring that your game state remains **atomic**. 

For example, in the pinata scenario mentioned earlier, the visual, movement, and hit detection of the pinata are handled entirely asynchronously. However, when rewarding the player by spawning a chest inventory with items inside, this inventory is "real" and is created from the main thread.

Let's see a sample of the code for the pinata using EpicAPI !

```java
// Some Hologram text
HoloComponent hpHoloComponent = HoloComponent.newBuilder()
	.location(baseLocation.clone().add(0,5,0))
	.text(() -> "§e§l" + currentHp + " / " + hpMax) // a text supplier to refresh pinata health display
	.build();
components.add(hpHoloComponent);

// The actual visual of the Pinata (composed of blocs)
List<BlockComponent> blockComponents = drawingLocations.stream()
	.map(drawLocation -> BlockComponent.newBuilder()
		.head(/*base 64 texture*/)
		.location(drawLocation)
		.build())
	.collect(Collectors.toList());
components.addAll(blockComponents);

// Build the structure object
Structure pinataStructure = Structure.newBuilder(BloodyPignata.getInstance())
	.components(components)
	.location(baseLocation)
	.onLeftClickAsync(player -> handleAttackPinata(player))
	.onRightClickAsync(player -> handleInteratPinata(player))
	.build();

// Enable to add it to the async context and automatically handle packet sending, interaction, ...
pinataStructure.enable();
```

EpicAPI expose a concept called a `Structure`, and is composed by a list of `Component`, and a component can be a lot of thing such as Entity, a block (using flying armorstand), a hologram etc.
You can assign interaction callback with the entire structure and EpicAPI will handle everything and send packets directly through netty thread behind the scene.

### -- THIS REPOSITORY IS STILL UNDER CONSTRUCTION
I'll share everything here when I have time.
Keep updated and give a star ☆ if you learned something interesting.



#### Credits

Thank to all BloodyBattle staff that was here for long years and special thanks to @MatD3mons who contributed to some plugins
