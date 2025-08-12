import discord

intents.messages = True
intents.guilds =
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
