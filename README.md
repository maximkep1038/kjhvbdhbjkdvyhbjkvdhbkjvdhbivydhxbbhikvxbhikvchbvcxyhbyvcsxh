import discord
from discord.ext import commands
from discord.ui import View, Select, Button

# ==== CONFIG ====
TOKEN = "MTQwNDg5OTg4NDY5NTU1NjE0OA.Gj_xrB.zZ_kQI-VJ3JBM59b4nWN9nS09MGquUrtAlh12o"
TICKET_CHANNEL_ID = 1404214235780743258  # Channel where the menu should be sent
TICKET_CATEGORY_ID = 1404214141400514631  # Category for new tickets
STAFF_ROLE_ID = 1404209643399413760       # Support role ID
LOGO_URL = "https://i.imgur.com/3ae8QPz.jpeg"   # Direct logo link
# ================

intents = discord.Intents.default()
intents.messages = True
intents.guilds = True
intents.members = True

bot = commands.Bot(command_prefix="!", intents=intents)


class TicketSelect(Select):
    def __init__(self):
        options = [
            discord.SelectOption(label="Purchase", description="Open a ticket for a purchase", emoji="💳"),
            discord.SelectOption(label="Support", description="Request help or support", emoji="🛠️"),
            discord.SelectOption(label="Exchange", description="Exchange a product", emoji="🔁"),
            discord.SelectOption(label="Product not received", description="If you did not receive your order", emoji="📦"),
            discord.SelectOption(label="Partnership", description="Request a partnership", emoji="🤝"),
        ]
        super().__init__(
            placeholder="Select a category...",
            min_values=1,
            max_values=1,
            options=options,
            custom_id="ticket_select_menu"
        )

    async def callback(self, interaction: discord.Interaction):
        category = discord.utils.get(interaction.guild.categories, id=TICKET_CATEGORY_ID)
        staff_role = interaction.guild.get_role(STAFF_ROLE_ID)
        overwrites = {
            interaction.guild.default_role: discord.PermissionOverwrite(view_channel=False),
            interaction.user: discord.PermissionOverwrite(view_channel=True, send_messages=True),
            staff_role: discord.PermissionOverwrite(view_channel=True, send_messages=True)
        }

        ticket_channel = await interaction.guild.create_text_channel(
            name=f"ticket-{interaction.user.name}",
            category=category,
            overwrites=overwrites
        )

        embed = discord.Embed(
            title="🎫 Your ticket has been created",
            description="Please describe your request in as much detail as possible. Our team will respond shortly.",
            color=0x00ff00
        )
        embed.set_thumbnail(url=LOGO_URL)
        await ticket_channel.send(content=f"{interaction.user.mention} Welcome!", embed=embed)

        await interaction.response.send_message(f"Ticket created: {ticket_channel.mention}", ephemeral=True)


class TicketMenu(View):
    def __init__(self):
        super().__init__(timeout=None)
        self.add_item(TicketSelect())


@bot.event
async def on_ready():
    bot.add_view(TicketMenu())  # Keep dropdown persistent
    print(f"✅ Bot {bot.user} is online.")

    channel = bot.get_channel(TICKET_CHANNEL_ID)
    if channel:
        embed = discord.Embed(
            title="SafeSupplier Tickets",
            description="If you need assistance, please select the appropriate category from the menu below.",
            color=0x00ff00
        )
        embed.set_thumbnail(url=LOGO_URL)
        await channel.send(embed=embed, view=TicketMenu())
        print(f"📨 Ticket menu sent to #{channel}.")
    else:
        print("❌ Ticket channel not found. Check the ID.")


bot.run(TOKEN)
