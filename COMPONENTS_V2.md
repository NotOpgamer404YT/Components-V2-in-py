# 📦 Discord.py — Components V2 Full Documentation

> A complete reference guide for building rich, modern Discord bot UIs using **Components V2** with `discord.py`.

---

## 📋 Table of Contents

1. [What is Components V2?](#what-is-components-v2)
2. [Requirements & Setup](#requirements--setup)
3. [Core Concept — LayoutView](#core-concept--layoutview)
4. [Component Reference](#component-reference)
   - [Container](#container)
   - [TextDisplay](#textdisplay)
   - [Separator](#separator)
   - [Section](#section)
   - [Thumbnail](#thumbnail)
   - [MediaGallery](#mediagallery)
   - [ActionRow](#actionrow)
   - [Button](#button)
   - [Select (Dropdown)](#select-dropdown)
5. [Building Layouts](#building-layouts)
6. [Interaction Handling](#interaction-handling)
7. [Full Examples](#full-examples)
   - [Avatar Command](#example-1-avatar-command)
   - [Help Menu with Dropdown](#example-2-help-menu-with-dropdown)
   - [User Info Card](#example-3-user-info-card)
   - [Confirmation Prompt](#example-4-confirmation-prompt)
8. [Common Patterns & Tips](#common-patterns--tips)
9. [Known Limitations](#known-limitations)

---

## What is Components V2?

**Components V2** is Discord's newer message component system that gives bots the ability to send richly structured, visually organized messages using a **layout-based approach** instead of plain embeds.

Key differences from classic embeds:

| Feature              | Classic Embeds         | Components V2            |
|----------------------|------------------------|--------------------------|
| Layout control       | Limited                | Full (containers, rows)  |
| Interactive elements | Buttons/selects outside| Inline inside containers |
| Images               | Thumbnail/image fields | MediaGallery component   |
| Text formatting      | Field-based            | Free-form TextDisplay    |
| Sections             | Not supported          | Section + Thumbnail      |

In `discord.py`, Components V2 is powered by `discord.ui.LayoutView` and the components inside `discord.ui`.

---

## Requirements & Setup

```bash
pip install discord.py
```

> ⚠️ Components V2 requires **discord.py 2.4+**. Make sure you are on the latest version.

Your bot needs these **intents** at minimum:

```python
intents = discord.Intents.default()
intents.message_content = True  # Required for prefix commands

bot = commands.Bot(command_prefix="!", intents=intents)
```

Imports you'll use most often:

```python
import discord
from discord.ext import commands
from discord import ui
from discord.ui import (
    LayoutView,
    Container,
    Section,
    TextDisplay,
    Separator,
    MediaGallery,
    Thumbnail,
    ActionRow,
    Button,
    Select,
)
from discord import SelectOption
```

---

## Core Concept — LayoutView

`LayoutView` is the **root** of every Components V2 message. Think of it as the blank canvas you paint your layout onto.

```python
class MyView(ui.LayoutView):
    def __init__(self):
        super().__init__(timeout=120)  # timeout in seconds, or None for no timeout
        
        # Build your layout here and add top-level items
        container = ui.Container()
        container.add_item(ui.TextDisplay("Hello, world!"))
        
        self.add_item(container)  # Add container to the LayoutView
```

Send it with:

```python
view = MyView()
await ctx.send(view=view)
```

### LayoutView Parameters

| Parameter | Type            | Description                                       |
|-----------|-----------------|---------------------------------------------------|
| `timeout` | `float \| None` | Seconds before the view stops listening. Default `180`. Use `None` for permanent. |

### Useful Methods

```python
self.clear_items()          # Remove all items from the view
self.add_item(component)    # Add a top-level component
await interaction.response.edit_message(view=self)  # Update the message
```

---

## Component Reference

### Container

A **Container** is a styled box that groups other components together. It's the main building block of any layout.

```python
container = ui.Container(
    *children,          # Pass components directly or use .add_item()
    accent_color=None,  # Optional: discord.Color or hex int for left border color
    spoiler=False,      # Optional: hide content behind spoiler blur
    id=None             # Optional: integer ID for referencing
)
```

**Adding children:**

```python
# Method 1: Pass in constructor
container = ui.Container(
    ui.TextDisplay("Line 1"),
    ui.TextDisplay("Line 2"),
)

# Method 2: add_item()
container = ui.Container()
container.add_item(ui.TextDisplay("Line 1"))
container.add_item(ui.TextDisplay("Line 2"))
```

**With accent color (colored left border):**

```python
container = ui.Container(
    ui.TextDisplay("This has a blue border!"),
    accent_color=discord.Color.blue()
)
```

**Spoiler container:**

```python
container = ui.Container(
    ui.TextDisplay("Hidden content!"),
    spoiler=True
)
```

---

### TextDisplay

Renders **markdown text** inside a layout. Supports full Discord markdown.

```python
ui.TextDisplay("Your text here")
```

**Markdown examples:**

```python
ui.TextDisplay("# Big Heading")
ui.TextDisplay("## Smaller Heading")
ui.TextDisplay("### Even Smaller")
ui.TextDisplay("**Bold** and *italic* and __underline__")
ui.TextDisplay("> This is a blockquote")
ui.TextDisplay("`inline code`")
ui.TextDisplay("-# Small/subtext line")
ui.TextDisplay("Regular paragraph text here.")
```

**Dynamic content:**

```python
ui.TextDisplay(f"**User:** {member.name}")
ui.TextDisplay(f"**ID:** `{member.id}`")
ui.TextDisplay(f"**Joined:** <t:{int(member.joined_at.timestamp())}:R>")
```

---

### Separator

Adds a **horizontal divider line** between components for visual separation.

```python
ui.Separator(
    spacing=ui.SeparatorSpacing.small,  # or .large
    visible=True                         # False = invisible spacing only
)
```

**Usage:**

```python
container = ui.Container(
    ui.TextDisplay("Section A"),
    ui.Separator(),                          # visible line, default spacing
    ui.TextDisplay("Section B"),
    ui.Separator(visible=False),             # invisible gap only
    ui.TextDisplay("Section C"),
    ui.Separator(spacing=ui.SeparatorSpacing.large),  # bigger gap
    ui.TextDisplay("Section D"),
)
```

---

### Section

A **Section** lays out text on the left and an accessory (like a `Thumbnail`) on the right side-by-side.

```python
ui.Section(
    *text_components,   # TextDisplay items for the left side
    accessory=...,      # A Thumbnail component on the right
    id=None
)
```

**Example:**

```python
section = ui.Section(
    ui.TextDisplay("### Welcome to the Server!"),
    ui.TextDisplay("Here's some info about what we do."),
    accessory=ui.Thumbnail(
        media=discord.UnfurledMediaItem(url="https://example.com/image.png"),
        description="Server logo"
    )
)
```

> 💡 `Section` is great for profile cards, help menus, and any layout where you want text next to an image.

---

### Thumbnail

A **small image** shown as an accessory inside a `Section`.

```python
ui.Thumbnail(
    media=discord.UnfurledMediaItem(url="https://..."),
    description="Alt text",   # Accessibility description
    spoiler=False
)
```

**Using a user's avatar:**

```python
ui.Thumbnail(
    media=discord.UnfurledMediaItem(url=member.display_avatar.url),
    description=f"{member.name}'s avatar"
)
```

**Using a bot's avatar:**

```python
ui.Thumbnail(
    media=discord.UnfurledMediaItem(url=bot.user.display_avatar.url),
    description="Bot icon"
)
```

> ⚠️ `Thumbnail` can **only** be used as an `accessory` in a `Section`, not standalone.

---

### MediaGallery

Displays **one or more images** in a gallery layout inside a container.

```python
gallery = ui.MediaGallery()
gallery.add_item(
    media="https://example.com/image.png",   # URL string
    description="Alt text",                  # Optional
    spoiler=False                            # Optional
)
```

**Using an attached file:**

```python
gallery = ui.MediaGallery()
gallery.add_item(media="attachment://avatar.png")
```

Then send the file alongside:

```python
await ctx.send(
    view=view,
    files=[discord.File(image_bytes, filename="avatar.png")]
)
```

**Multiple images:**

```python
gallery = ui.MediaGallery()
gallery.add_item(media="https://example.com/image1.png", description="Image 1")
gallery.add_item(media="https://example.com/image2.png", description="Image 2")
gallery.add_item(media="https://example.com/image3.png", description="Image 3")
```

> 💡 Up to **10 images** can be added to a single `MediaGallery`.

---

### ActionRow

A **horizontal row** that holds interactive elements like Buttons and Selects.

```python
row = ui.ActionRow(
    ui.Button(label="Click me", style=discord.ButtonStyle.primary),
    ui.Button(label="Another", style=discord.ButtonStyle.secondary),
)
```

Or build it step by step:

```python
row = ui.ActionRow()
row.add_item(ui.Button(label="Yes", style=discord.ButtonStyle.success))
row.add_item(ui.Button(label="No", style=discord.ButtonStyle.danger))
```

> ⚠️ An `ActionRow` can hold **up to 5 Buttons** OR **1 Select menu**, not both.

---

### Button

A clickable **button** in an `ActionRow`.

```python
ui.Button(
    label="Click Me",                       # Button text
    style=discord.ButtonStyle.primary,      # Style (see below)
    custom_id="my_button",                  # For callback buttons
    url="https://...",                      # For link buttons (ButtonStyle.link only)
    emoji="✅",                             # Optional emoji
    disabled=False,                         # Grey out button
    row=None
)
```

**Button Styles:**

| Style                          | Appearance   | Use Case              |
|--------------------------------|--------------|-----------------------|
| `discord.ButtonStyle.primary`  | Blue         | Main action           |
| `discord.ButtonStyle.secondary`| Grey         | Secondary/neutral     |
| `discord.ButtonStyle.success`  | Green        | Confirm/positive      |
| `discord.ButtonStyle.danger`   | Red          | Delete/destructive    |
| `discord.ButtonStyle.link`     | Grey + arrow | External URL redirect |

**Callback button (responds to clicks):**

```python
btn = ui.Button(label="Confirm", style=discord.ButtonStyle.success, custom_id="confirm_btn")

async def on_confirm(interaction: discord.Interaction):
    await interaction.response.send_message("Confirmed!", ephemeral=True)

btn.callback = on_confirm
```

**Link button (no callback needed):**

```python
btn = ui.Button(
    label="Visit Website",
    style=discord.ButtonStyle.link,
    url="https://discord.com"
)
```

---

### Select (Dropdown)

A **dropdown menu** users can select option(s) from.

```python
ui.Select(
    placeholder="Choose an option...",
    min_values=1,
    max_values=1,
    options=[
        discord.SelectOption(label="Option A", value="a", description="First option", emoji="🔴"),
        discord.SelectOption(label="Option B", value="b", description="Second option", emoji="🟢"),
    ],
    custom_id="my_select",
    disabled=False
)
```

**SelectOption Parameters:**

| Parameter     | Description                        |
|---------------|------------------------------------|
| `label`       | Visible text                       |
| `value`       | Internal value sent to callback    |
| `description` | Small subtext below label          |
| `emoji`       | Emoji before label                 |
| `default`     | Pre-selected if `True`             |

**Handling a select callback:**

```python
self.dropdown = ui.Select(
    placeholder="Pick a category...",
    options=[
        discord.SelectOption(label="Moderation", value="mod"),
        discord.SelectOption(label="Fun", value="fun"),
    ]
)
self.dropdown.callback = self.on_select

async def on_select(self, interaction: discord.Interaction):
    chosen = self.dropdown.values[0]  # Get selected value
    await interaction.response.edit_message(...)
```

---

## Building Layouts

### Structure Overview

```
LayoutView
└── Container
    ├── TextDisplay
    ├── Separator
    ├── Section
    │   ├── TextDisplay (left side)
    │   └── Thumbnail (right side, accessory)
    ├── MediaGallery
    │   └── image items
    ├── Separator
    └── ActionRow
        ├── Button
        └── Button
```

### Nesting Rules

| Parent      | Can Contain                                      |
|-------------|--------------------------------------------------|
| `LayoutView`| `Container` (top-level only)                     |
| `Container` | `TextDisplay`, `Separator`, `Section`, `MediaGallery`, `ActionRow` |
| `Section`   | `TextDisplay` (left), `Thumbnail` (accessory)    |
| `ActionRow` | `Button` (up to 5) OR `Select` (1 only)          |
| `MediaGallery` | Image items via `.add_item()`                 |

> ⚠️ You **cannot** put a `Container` inside another `Container`.

---

## Interaction Handling

### Restricting to Command Author

Override `interaction_check` to prevent other users from using your buttons/selects:

```python
class MyView(ui.LayoutView):
    def __init__(self, author: discord.Member):
        super().__init__(timeout=60)
        self.author = author

    async def interaction_check(self, interaction: discord.Interaction) -> bool:
        if interaction.user.id != self.author.id:
            await interaction.response.send_message(
                "This isn't your menu!", ephemeral=True
            )
            return False
        return True
```

### Responding to Interactions

```python
# Send a new message
await interaction.response.send_message("Done!", ephemeral=True)

# Edit the current message
await interaction.response.edit_message(view=self)

# Defer (use when processing takes time)
await interaction.response.defer()
# ... do work ...
await interaction.followup.send("Finished!")
```

### Rebuilding the Layout on Interaction

```python
async def on_button_click(self, interaction: discord.Interaction):
    # Clear old layout
    self.clear_items()
    
    # Build new layout
    new_container = ui.Container(
        ui.TextDisplay("# Updated!"),
        ui.TextDisplay("The layout was rebuilt."),
    )
    self.add_item(new_container)
    
    # Push the edit
    await interaction.response.edit_message(view=self)
```

---

## Full Examples

### Example 1: Avatar Command

Displays a user's avatar in a gallery with format download buttons.

```python
import discord
from discord.ext import commands
from discord import ui
from typing import Optional
import requests
from io import BytesIO


class Avatar(commands.Cog):
    def __init__(self, bot):
        self.bot = bot

    @commands.command(name="avatar", aliases=["av", "pfp"])
    @commands.cooldown(1, 5, commands.BucketType.user)
    async def avatar_command(self, ctx, member: Optional[discord.Member] = None):
        member = member or ctx.author
        avatar = member.display_avatar

        is_animated = avatar.is_animated()
        fmt = "gif" if is_animated else "png"
        filename = f"avatar.{fmt}"

        response = requests.get(avatar.replace(size=1024, format=fmt).url)
        avatar_data = BytesIO(response.content)

        class AvatarView(ui.LayoutView):
            def __init__(self):
                super().__init__()

                png_url  = member.display_avatar.replace(size=1024, format="png").url
                jpg_url  = member.display_avatar.replace(size=1024, format="jpg").url
                webp_url = member.display_avatar.replace(size=1024, format="webp").url

                gallery = ui.MediaGallery()
                gallery.add_item(media=f"attachment://{filename}")

                btn_png  = ui.Button(label="PNG",  style=discord.ButtonStyle.link, url=png_url)
                btn_jpg  = ui.Button(label="JPG",  style=discord.ButtonStyle.link, url=jpg_url)
                btn_webp = ui.Button(label="WEBP", style=discord.ButtonStyle.link, url=webp_url)
                button_row = ui.ActionRow(btn_png, btn_jpg, btn_webp)

                avatar_type = "Animated (GIF)" if is_animated else "Static"

                container = ui.Container(
                    ui.TextDisplay(f"# {member.name}'s Avatar"),
                    ui.Separator(),
                    ui.TextDisplay(f"**User:** {member.mention}"),
                    ui.TextDisplay(f"**ID:** `{member.id}`"),
                    ui.TextDisplay(f"**Type:** {avatar_type}"),
                    ui.TextDisplay("**Size:** 1024×1024"),
                    gallery,
                    ui.Separator(),
                    ui.TextDisplay("**Download formats:**"),
                    button_row,
                )
                self.add_item(container)

        await ctx.send(
            view=AvatarView(),
            files=[discord.File(avatar_data, filename=filename)]
        )


async def setup(bot):
    await bot.add_cog(Avatar(bot))
```

---

### Example 2: Help Menu with Dropdown

Interactive category browser using a Select menu.

```python
import discord
from discord.ext import commands
from discord import ui, SelectOption

COMMANDS = {
    "Moderation": ["`ban`", "`kick`", "`mute`", "`warn`", "`purge`"],
    "Utility":    ["`avatar`", "`userinfo`", "`serverinfo`", "`ping`"],
    "Fun":        ["`8ball`", "`meme`", "`joke`", "`rps`"],
    "Economy":    ["`balance`", "`daily`", "`work`", "`shop`"],
}


class HelpView(ui.LayoutView):
    def __init__(self, bot: commands.Bot, author: discord.Member):
        super().__init__(timeout=120)
        self.bot = bot
        self.author = author
        self._build_home()

    def _build_home(self):
        self.clear_items()

        self.dropdown = ui.Select(
            placeholder="Select a category...",
            options=[
                SelectOption(
                    label=cat,
                    description=f"{len(cmds)} commands",
                    value=cat
                )
                for cat, cmds in COMMANDS.items()
            ]
        )
        self.dropdown.callback = self.on_select

        section = ui.Section(
            ui.TextDisplay("### 🤖 Bot Help"),
            ui.TextDisplay("Use the dropdown below to browse commands by category."),
            accessory=ui.Thumbnail(
                media=discord.UnfurledMediaItem(url=self.bot.user.display_avatar.url),
                description="Bot avatar"
            )
        )

        container = ui.Container(
            section,
            ui.Separator(),
            ui.TextDisplay(f"-# {len(COMMANDS)} categories available"),
            ui.ActionRow(self.dropdown)
        )
        self.add_item(container)

    async def interaction_check(self, interaction: discord.Interaction) -> bool:
        if interaction.user.id != self.author.id:
            await interaction.response.send_message(
                "Run the `help` command yourself to use this menu.", ephemeral=True
            )
            return False
        return True

    async def on_select(self, interaction: discord.Interaction):
        category = self.dropdown.values[0]
        cmds = COMMANDS.get(category, [])
        self.clear_items()

        # Re-create dropdown to keep it in the updated view
        self.dropdown = ui.Select(
            placeholder="Select a category...",
            options=[
                SelectOption(
                    label=cat,
                    description=f"{len(c)} commands",
                    value=cat,
                    default=(cat == category)
                )
                for cat, c in COMMANDS.items()
            ]
        )
        self.dropdown.callback = self.on_select

        section = ui.Section(
            ui.TextDisplay(f"### 📂 {category}"),
            ui.TextDisplay("  ".join(cmds)),
            accessory=ui.Thumbnail(
                media=discord.UnfurledMediaItem(url=self.bot.user.display_avatar.url),
                description=f"{category} icon"
            )
        )

        container = ui.Container(
            section,
            ui.Separator(),
            ui.TextDisplay("-# Use the dropdown to switch categories."),
            ui.ActionRow(self.dropdown)
        )
        self.add_item(container)
        await interaction.response.edit_message(view=self)


class HelpCog(commands.Cog):
    def __init__(self, bot):
        self.bot = bot

    @commands.command(name="help")
    async def help_command(self, ctx):
        await ctx.send(view=HelpView(self.bot, ctx.author))


async def setup(bot):
    await bot.add_cog(HelpCog(bot))
```

---

### Example 3: User Info Card

A clean profile card using Section + Thumbnail.

```python
import discord
from discord.ext import commands
from discord import ui
from typing import Optional


class UserInfo(commands.Cog):
    def __init__(self, bot):
        self.bot = bot

    @commands.command(name="userinfo", aliases=["ui", "whois"])
    async def userinfo(self, ctx, member: Optional[discord.Member] = None):
        member = member or ctx.author

        joined   = f"<t:{int(member.joined_at.timestamp())}:R>"
        created  = f"<t:{int(member.created_at.timestamp())}:R>"
        roles    = [r.mention for r in member.roles if r.name != "@everyone"]
        top_role = member.top_role.mention if member.top_role.name != "@everyone" else "None"
        status   = str(member.status).capitalize()

        class UserInfoView(ui.LayoutView):
            def __init__(self):
                super().__init__()

                section = ui.Section(
                    ui.TextDisplay(f"# {member.display_name}"),
                    ui.TextDisplay(f"**Username:** {member.name}"),
                    ui.TextDisplay(f"**ID:** `{member.id}`"),
                    ui.TextDisplay(f"**Status:** {status}"),
                    accessory=ui.Thumbnail(
                        media=discord.UnfurledMediaItem(url=member.display_avatar.url),
                        description=f"{member.name}'s avatar"
                    )
                )

                container = ui.Container(
                    section,
                    ui.Separator(),
                    ui.TextDisplay(f"**Joined Server:** {joined}"),
                    ui.TextDisplay(f"**Account Created:** {created}"),
                    ui.TextDisplay(f"**Top Role:** {top_role}"),
                    ui.TextDisplay(
                        f"**Roles ({len(roles)}):** {', '.join(roles[:5]) or 'None'}"
                        + (" ..." if len(roles) > 5 else "")
                    ),
                    accent_color=member.color if member.color.value else discord.Color.blurple()
                )
                self.add_item(container)

        await ctx.send(view=UserInfoView())


async def setup(bot):
    await bot.add_cog(UserInfo(bot))
```

---

### Example 4: Confirmation Prompt

A yes/no prompt with dynamic layout updates on button click.

```python
import discord
from discord.ext import commands
from discord import ui


class ConfirmView(ui.LayoutView):
    def __init__(self, author: discord.Member, action: str):
        super().__init__(timeout=30)
        self.author = author
        self.action = action
        self._build_prompt()

    def _build_prompt(self):
        self.clear_items()

        btn_yes = ui.Button(label="✅ Confirm", style=discord.ButtonStyle.success, custom_id="yes")
        btn_no  = ui.Button(label="❌ Cancel",  style=discord.ButtonStyle.danger,  custom_id="no")

        btn_yes.callback = self.on_confirm
        btn_no.callback  = self.on_cancel

        container = ui.Container(
            ui.TextDisplay(f"## ⚠️ Are you sure?"),
            ui.Separator(),
            ui.TextDisplay(f"You are about to: **{self.action}**"),
            ui.TextDisplay("-# This action cannot be undone."),
            ui.Separator(),
            ui.ActionRow(btn_yes, btn_no),
            accent_color=discord.Color.yellow()
        )
        self.add_item(container)

    async def interaction_check(self, interaction: discord.Interaction) -> bool:
        if interaction.user.id != self.author.id:
            await interaction.response.send_message("Not your prompt!", ephemeral=True)
            return False
        return True

    async def on_confirm(self, interaction: discord.Interaction):
        self.clear_items()
        container = ui.Container(
            ui.TextDisplay("## ✅ Confirmed!"),
            ui.TextDisplay(f"Action executed: **{self.action}**"),
            accent_color=discord.Color.green()
        )
        self.add_item(container)
        await interaction.response.edit_message(view=self)

    async def on_cancel(self, interaction: discord.Interaction):
        self.clear_items()
        container = ui.Container(
            ui.TextDisplay("## ❌ Cancelled"),
            ui.TextDisplay("No changes were made."),
            accent_color=discord.Color.red()
        )
        self.add_item(container)
        await interaction.response.edit_message(view=self)


class ModerationCog(commands.Cog):
    def __init__(self, bot):
        self.bot = bot

    @commands.command(name="nuke")
    @commands.has_permissions(administrator=True)
    async def nuke(self, ctx):
        view = ConfirmView(ctx.author, f"Nuke #{ctx.channel.name}")
        await ctx.send(view=view)


async def setup(bot):
    await bot.add_cog(ModerationCog(bot))
```

---

## Common Patterns & Tips

### ✅ Always rebuild the dropdown when editing the message

When you call `self.clear_items()` and rebuild, you must re-add your Select/Button items and re-assign their callbacks — otherwise they disappear.

```python
async def on_select(self, interaction):
    self.clear_items()
    
    # Re-create the dropdown
    self.dropdown = ui.Select(placeholder="...", options=[...])
    self.dropdown.callback = self.on_select  # ← Don't forget this!
    
    new_container = ui.Container(
        ui.TextDisplay("New content"),
        ui.ActionRow(self.dropdown)
    )
    self.add_item(new_container)
    await interaction.response.edit_message(view=self)
```

### ✅ Using attachment:// for local images

When using images from files (not URLs), use `attachment://filename`:

```python
gallery = ui.MediaGallery()
gallery.add_item(media="attachment://image.png")

await ctx.send(
    view=view,
    files=[discord.File(BytesIO(data), filename="image.png")]
)
```

### ✅ Accent colors on containers

Add a colored left border to any container easily:

```python
ui.Container(
    ...,
    accent_color=discord.Color.from_rgb(255, 100, 0)  # Custom orange
)

# Or use a member's role color:
ui.Container(..., accent_color=member.color)
```

### ✅ Timestamps in TextDisplay

Use Discord's native timestamp formatting for live-updating times:

```python
import datetime

ts = int(datetime.datetime.utcnow().timestamp())
ui.TextDisplay(f"Now: <t:{ts}:F>")   # Full date+time
ui.TextDisplay(f"Relative: <t:{ts}:R>")  # "3 minutes ago"
ui.TextDisplay(f"Date: <t:{ts}:D>")  # Date only
```

### ✅ Ephemeral error messages during interaction_check

```python
async def interaction_check(self, interaction: discord.Interaction) -> bool:
    if interaction.user.id != self.author.id:
        await interaction.response.send_message(
            "This menu isn't for you!", ephemeral=True
        )
        return False
    return True
```

### ✅ Disable buttons after use

```python
async def on_button(self, interaction: discord.Interaction):
    self.clear_items()
    
    disabled_btn = ui.Button(label="Used", style=discord.ButtonStyle.secondary, disabled=True)
    container = ui.Container(
        ui.TextDisplay("Done!"),
        ui.ActionRow(disabled_btn)
    )
    self.add_item(container)
    await interaction.response.edit_message(view=self)
```

---

## Known Limitations

| Limitation | Details |
|------------|---------|
| No nested Containers | You cannot put a `Container` inside another `Container` |
| Thumbnail is Section-only | `Thumbnail` can only be used as an `accessory` in `Section` |
| ActionRow limit | Max 5 Buttons OR 1 Select per ActionRow |
| MediaGallery limit | Up to 10 images per gallery |
| Components V2 messages | Cannot mix with classic embeds in the same send |
| LayoutView top-level | Only `Container` can be added directly to `LayoutView` |
| Select in ActionRow | A Select must be the **only** item in its ActionRow |

---

> 📝 **Tip:** Always test your layouts in a development server first. Discord sometimes updates component rendering behavior, and layouts that look fine in one Discord client version may appear slightly different in another.

---

*Made with ❤️ for discord.py bot developers*
