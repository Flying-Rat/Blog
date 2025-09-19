---
title: "Godot Itch Plugin"
date: 2025-09-19T10:00:00+02:00
authors:
  - Jan Mesarč
  - Jakub Hubáček
categories:
  - godot
  - tools
tags:
  - godot
  - gamedev
  - itch.io
  - plugin
toc: true
---

## Introduction
What is this about? Well, we are two friends and colleagues Jan and Jakub and we were lucky. We just successfully finished one project and what now? The new project will start soon, and we need to fill a short period of time between. We are big fans of Godot Engine and we suggest using this time to do something interesting for us, something new what we can learn, and it can be helpful for Godot community.

<div style="text-align:center">
  <img src="https://upload.wikimedia.org/wikipedia/commons/5/5a/Godot_logo.svg" alt="Godot Logo" style="width:50%;max-width:50%;height:auto;" />
</div>


We set up a meeting with our CTO and presented our ideas. Can we take a week? What about dialog system? No? Then we can try optimization toolkit, handy tools for checking assets or possibly issue in project and performance. Maybe we can create plugin? Martin our CTO smiled and said that's sounds good. You have 24 hours to show off. 

Alright, challenge accepted. 24 hours is not too much time, so we are thinking about our ideas and suggestions. We picked creating plugin for **Itch.io**. Why? We wanted to start with something small. **Itch** API is minimal, and we know we must dig up path through the plugin creation. Let's look on problem what we wanted to resolve. Tick tack

## Problem Statement
Game development is very strictly tied to specific stores when it comes to game publishing, there is Steam, Epic, GoG and for small indie developers there is Itch. It's common practice to use features from platform store - unlock achievements, use leaderboards, verify game purchase. 

For Steam there is a Godot addon called [Godot Steam](https://godotsteam.com/), but for [itch.io](https://itch.io/) there isn't any now. This is our opportunity to create an addon which brings more features to the community, a community for which itch.io is a major publishing platform. You may object - Godot games are mostly small and free- and would be right, but that's fine by us.

Our goal is to create a developer-friendly **Godot** addon which can easily provide **itch.io** API features. Our top one priority feature is **purchasing verification**, a simple way for developers to verify player has bought the game. As **itch.io** API slowly grows, new functions will be added, we can be there to support them as well.

## Research
It's a little farfetched to call it research, just imagine us reading through the Godot addon documentation and itch.io API documentation, finding out what is possible now and how can we use it, what I want to pinpoint here is:

- We are used to working with different SDKs libraries, our original intent was to download library from itch.io and use it
- itch.io does not provide SDK, they use public API which you can query using http calls
- itch.io allows checking game purchase by username, but that's weak verification
- it's possible to use OAuth to verify player, but that's sounds too intrusive
- itch.io provides his own launcher, but since it's not widely used, we are not targeting this use case

What we choose to support is **purchase verification** by **download key** - a unique URL sent to each buyer after purchase. It’s simple, secure enough, and easy to implement. Tick tack.

## Proposal
The clock is ticking, mission is clear, build a plugin that any Godot developer can simply hook into their project and use right away.

- Create Godot Itch v1.0 plugin
- Implement functionality for purchase verification via download key
- Create and provide user friendly sample project for developers
- Publish it to Godot Asset Library
- Profit! Eternal glory! (Yes, addon is for free)

## How we selected technology
We started out with only a rough idea of the options for creating plugins in Godot — we’d used Editor Plugins before, and **GDExtension** had been on our bucket list for a while. So we decided to push ourselves: we explored both the easy, editor-focused route and the native GDExtension route to see which fit best.

### Editor plugin
A faster way to create an addon for editor with minimal setup, just look at this guide: https://docs.godotengine.org/en/4.4/tutorials/plugins/editor/making_plugins.html

Creating an addon is really easy, you can extend your editor or create a singleton to use and all just with gdscript. Thats the main benefit.

#### Advantages of Editor plugin
- It's easy to create your own editor plugin - with gdscript which you are already using for gameplay code anyway.
- It has tun of options - you can create your own data table editor or resource manager or project settings with guide and examples. 
- Compile time is amazing, especially since there is no compile time - it's ready for publishing. 

#### Setup
Did you know there is a button for creating editor plugin within Godot editor?
Just go to Project->Project Settings->Plugins and there it is, in the corner of the menu there is button "Create Plugin". Easy right?

#### Implementation
We can create a singleton and instance it each time a game is played. 
Even more we can create a nicely looking menu which we can attach into Project Settings!

![Godot Itch Plugin Guide](../assets/images/godotItchPlugin/godot_itch_plugin_guide.png)

Did you know you add a button literally everywhere? Thats unique about Godot Editor, every part can be modified, every element can be altered. This of course can lead to crashes and functionality issues if you remove essential elements. But you can do it, which is impressive! 

For example, the menu you can add menu you see above into project settings as another panel (next to the Plugins, Autoloads) with this code in your plugin.gdscript. It's amazing how flexible the engine is 😲

```
@tool
extends EditorPlugin

func _enter_tree() -> void:
  _create_itch_project_settings_panel()

func _create_itch_project_settings_panel() -> void:
  # Load a styled settings scene and use it as the panel
  var scene = load("res://addons/godot_itch/settings/itch_settings_panel.tscn")
  if scene:
    # Create an instance of     
    var itch_panel = scene.instantiate()

    # Adds a custom panel into Godot Editor -> Project Settings panel
    add_control_to_container(CustomControlContainer.CONTAINER_PROJECT_SETTING_TAB_RIGHT, itch_panel)
```

#### We prefer the hard way
We chose not to use Editor Plugin (gdscript) approach, because we are mainly developers and we are heavily interested in gdextension and its possibilities. We can combine approaches and use both, gdextension and editor plugin in GDScript and what it's our plan.

### Why GDExtension?
We chose **GDExtension**, because it is a Godot-Specific technology and feature that allows native code integration without recompiling the engine. It was released with Godot 4 version, and it is still marked as an experimental feature.

#### Benefits

- No need to compile Godot Engine itself
- Gives you access to most of the API available to GDScript or C#
- Allowing you to code game logic
- Ideal if you need high-performance code
- Support for more programming languages besides C++ (Go, Rust, Odin, Swift, D, etc.)

We used C++ for this project and following the official GDExtension setup guide in [Godot documentation](https://docs.godotengine.org/en/stable/tutorials/scripting/gdextension/gdextension_cpp_example.html#). We can briefly go over the setup.

#### Project Setup

You need these tools:
- Godot 4 executable
- C++ compiler
- SCons build tool (SCons: A software construction tool - SCons)
- godot-cpp GitHub - godotengine/godot-cpp: C++ bindings for the Godot script API

Create these folders for your **GDExtension**:

- `demo/` - Godot test project
- `godot-cpp/` - C++ bindings (Git submodule)
- `src/` - GDExtension source code

In the demo folder, create a new Godot project and set up a basic scene. The godot-cpp folder is a Git submodule. The src folder holds the code where the main magic happens.

#### Integrating the **itch.io** API
We originally targeted the **download key verification** endpoint. But **OAuth** tokens available to the game do not have the required scope to call download key endpoints. Those endpoints require a server-side **API key**. Because of that, the extension currently focuses on **OAuth**-authenticated endpoints, starting with the **user identity endpoint**: `GET /me`.

Check the Serverside API reference here: [Serverside API reference - itch.io](https://itch.io/docs/api/serverside). 

Our implementation of **GDExtension** called **GodotItch** contains the C++ code for the **Godot Itch** extension.

- `godotitch.h` / `godotitch.cpp`: Main class that talks to the Itch.io API (HTTP requests, signals, helper methods).
- `register_types.h` / `register_types.cpp`: Registers the extension’s classes with Godot.

Current endpoint implemented: `GET /me`. What it does?
- Queries itch.io for the currently authenticated user (via OAuth token).
- Runs asynchronously and emits signals when done.

Flow
[Start] → [Ensure HTTPRequest/scene init] → [Ensure OAuth token present] → [Call /me] → [Handle response via signals]

Signals
- api_response("get_me", data) on success (data.user contains id, username, display_name, ...)
- api_error("get_me", error_message, response_code) on failure

How to call it (typical usage)
1) Initialize the extension in your scene so HTTPRequest exists and signals are connected.
2) Ensure an OAuth token is available (e.g., obtained via the OAuth example or when launched from the itch app).
3) Call Itch.get_me() and handle the signals.

Example (GDScript):
```
# ...existing code...
# Ensure this runs inside a node in your scene:
Itch.initialize_with_scene(self)

# Connect once (idempotent in example code)
if not Itch.api_response.is_connected(_on_api_response):
    Itch.api_response.connect(_on_api_response)
if not Itch.api_error.is_connected(_on_api_error):
    Itch.api_error.connect(_on_api_error)

# Optional: check token availability
var auth = Itch.get_auth()
if auth and auth.has_method("get_oauth_token"):
    var tok := str(auth.get_oauth_token())
    if tok.length() == 0:
        print("No OAuth token set. Use the OAuth example or launch via the itch app.")

# Call the endpoint
Itch.get_me()

func _on_api_response(endpoint: String, data: Dictionary) -> void:
    if endpoint == "get_me":
        var user := data.get("user", {})
        print("User:", user.get("id", ""), user.get("username", ""), user.get("display_name", ""))

func _on_api_error(endpoint: String, error_message: String, response_code: int) -> void:
    push_error("[%s] (%d) %s" % [endpoint, response_code, error_message])
# ...existing code...
```

See? Nothing special. Notes:

- Download key verification is not callable with the game’s OAuth token; use a server-side API key if you need to validate download keys.
- The example scene demonstrating this flow is addons/godot_itch/example/example_get_me.gd.
- You can explore the full sample project here: https://github.com/Flying-Rat/GodotItch/tree/main/Samples/GodotItchExtension


## Development Journey
⏰ Tick tock! Just like kings in medieval times, we had to divide and conquer to achieve our goal.

- Jakub started working on the Editor Plugin using only GDScript.  
- Jan dove into GDExtension, which allows writing code in C++.  

### Let's prototype! 
- The developer saves the API key and game ID into the project settings.  
- The player only sees a download key input field—and once entered, they’re verified!  

🎉 I’m happy to announce that within just 24 hours we built a working prototype, exactly as we set out to do! 🎉

We were so excited about our progress that we didn’t wait and created a draft for the Godot Asset Library. The idea was to get feedback as soon as possible so we could address it and resubmit quickly, since the review process can take some time.

![Godot Itch Plugin AssetLib Creation](../assets/images/godotItchPlugin/godot_itch_plugin_creation_asset_lib.png)

### Happy Beginnings
We proved that we can deliver a prototype - something essential for continuing work on the project.

Over the next few days, we put together a strict plan of features we want to support:

- Robust subsystem friendly library
- Various endpoints supported
- Simple examples
- Native and markdown documentation
- Caching with lifetime
- Submit to Godot Asset Library

Subsystems for various areas (Users, Games, Purchases) make sense and its what we are used to in other platform libraries like Steam, we want something similar here, code architecture matters!

Also did you know you can create native documentation for GDExtension? So you can use it same way as you native documentation within editor, how cool is that. 

Nobody can stop us—we know what we’re doing, and nothing could possibly go wrong. Right? …Right?! Well.

### Just a flesh wound
Remember when I mentioned we were using the game ID and API key for purchase-key verification? Our idea was simple: put the API key into project settings and ship it with the game. It worked… but it was a terrible idea — completely insecure and, honestly, probably against itch.io’s rules.

![Godot Itch Plugin API Key](../assets/images/godotItchPlugin/godot_itch_api_key.png)

Why is this a bad idea? The API key itch.io gives you is unlimited. As a developer, you’d paste it into the game and ship it — but then every player could grab your unlimited API key (all they’d need is your username, which is easy to find).

How bad could it be? Really bad. With an unlimited API key, an attacker could delete content from your itch.io account — but that’s not even the worst part. Imagine someone downloading your build, sneaking in a tiny Trojan DLC, and uploading it to your itch.io. Definitely not what we signed up for!

So, what now — call it even and forget about it?

### No time for surrender
It’s clear we **cannot use the API key as we originally intended**. We even considered adding password protection for the API key, but we dismissed that option — it’s critical not to share this key. It’s just too risky.

Luckily, we discovered another option: OAuth! OAuth can usually grant you a token that’s valid for a limited time. Have you ever logged into a shop using your Google, Facebook, or Apple account? If so, you already have an idea of how the OAuth process works.

With renewed hope, we jumped back into the code and started rewriting our plugin. We can’t use the API key, but we can use an OAuth token - and that’s amazing because these tokens intentionally have a limited lifetime. 

How dangerous is it to share your API key? Similarly to sharing your password ⬇️

![Itch.io API Key Doc](../assets/images/godotItchPlugin/godot_itch_api_key_doc.png)

What is the difference between OAuth token and API Key?

First API Key is a long-lasting key that gives full access to your account. If someone gets it, they can do anything. It’s powerful but risky to share. 

OAuth token. For itch.io it is not JWT token, but it is more like API Key, but with limited access. It is not like a regular token.  The scope is limited to user information only. That's bad, we cannot list purchased games, and we cannot verify download keys. Right now, there’s no user-friendly way for developers to verify that a user has purchased a game using the itch.io API.

### Imperfections are fine

The reality of limited scopes hits hard — this block 90% of the features we planned to support. 
Did I mention we pay attention to code architecture? Since we started with the idea of having different subsystems.

```
- src/
  - auth_subsystem/
    - `auth_subsystem.cpp`
    - `auth_subsystem.h`
  - core_subsystem/
    - `core_subsystem.cpp`
    - `core_subsystem.h`
  - games_subsystem/
    - `games_subsystem.cpp`
    - `games_subsystem.h`
  - user_subsystem/
    - `user_subsystem.cpp`
    - `user_subsystem.h`
```

You probably get the idea - each subsystem is responsible for a specific part of the platform’s functionality. The User subsystem provides information about the user, and Games provides information about the game itself. There is templated class with a lifetime management functionality. Yes, I like it a lot 😁

<div style="text-align:center">
  <img src="https://media2.giphy.com/media/v1.Y2lkPTc5MGI3NjExN3R2bnZsaGc3YnBud2MybTRnMXA3OW5yemRjODNxMmpwcXd4cmdsNCZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw/Qo3qovmbqaKT6/giphy.gif" alt="Giphy" />
</div>

Guess what, we only need two of them for now!💣😥 I don’t consider this time wasted; it was educational, and we’ll be reusing it, so its fine.



Meanwhile, we did our best — we implemented the OAuth verification process, which gives us a limited API key. With this key, we can still request basic user information, like username, display name, and user cover image.

There is a use case for our plugin. It’s not exactly what we originally intended, but it’s not useless either.

## Publish to Godot Asset Library
From the start, our plan was to share this with the community, that's the original goal. I already mentioned we published the first version early, so early we didn't know what fatal issue there is, but we didn't talk much about the process.

Publishing on Godot Asset Library was surprisingly straightforward; you just create an account and hit Submit Asset. Before hitting publish, there’s a handy checklist to follow: https://docs.godotengine.org/en/latest/community/asset_library/submitting_to_assetlib.html

Here are a few highlights:

- Pick a clear name, category, and license 
- Keep structure: Project/addons/my_addon_name  
- Verify Godot version
- Include examples
- Keep it clean

The submission process is smooth, so there is nothing to stress about.

And the best part: **our plugin was accepted! 🚀** It took us 3 days, which is common time according to the community.

You can try it right now, just open your **Godot** editor and search for **Godot Itch**!

![Godot Editor AssetLib Preview](../assets/images/godotItchPlugin/godot_itch_asset_lib.png)

## Future roadmap 
Sitting and sobbing in the corner is not an option. Just an hour ago, we sent an email to itch.io asking about expanding the scopes for their API endpoints. Once they get back to us, we’ll have a better idea of their roadmap and can plan accordingly.

In the meantime, we’re brainstorming features for future releases. Here are a few ideas: 

- Itch.io launcher support
- Key verification toolkit
- Publishing toolkit
- Release to Godot Beta Store

Since we now know how to extend editor functionality, it's easy to imagine a whole new menu or panel with various itch.io-related features -managing GitHub Actions, creating packages, managing published versions, or checking how many keys have been claimed. All of this would be on the developer side only, so there’s no risk of the API key leaking, and it could still provide a lot of useful functionality.

## Conclusion
We made something that can actually help developers, even if it’s just in the way we first imagined.
We pushed ourselves with tight deadlines, dove into new territory, and managed to come out with only a few small bruises.
Along the way, we finally got to explore Editor Plugins and GDExtensions, which had been on our bucket list for ages.
Most importantly, we built this for the Godot community — a community of passionate game devs we love being part of.

## Feedback
Do you have questions, feedback, or ideas for what this could become next? Drop us a comment, share your thoughts, or just let us know if it’s been useful — your input means a lot and helps us figure out what to pay attention to.

## Links
Asset Library: [Godot Itch - Godot Asset Library](https://godotengine.org/asset-library/asset/4302)
Github repository: [GitHub - Flying-Rat/GodotItch: A Godot plugin for itch.io](https://github.com/Flying-Rat/GodotItch)
Github issues: [Flying-Rat/GodotItch](https://github.com/Flying-Rat/GodotItch/issues)
