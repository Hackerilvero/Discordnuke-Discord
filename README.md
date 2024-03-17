import discord
import asyncio

class DiscordAdminTool:
    def __init__(self):
        self.logo = """
\033[92m         _____  _               _   _       _             
        |  __ \\(_)             | \\ | |     | |            
        | |  | |_ ___  ___ ___ |  \\| |_   _| | _____ _ __ 
        | |  | | / __|/ __/ _ \\| . ` | | | | |/ / _ \\ '__|
        | |__| | \\__ \\ (_| (_) | |\\  | |_| |   <  __/ |   
        |_____/|_|___/\\___\\___/|_| \\_|\\__,_|_|\\_\\___|_|   
\033[0m\033[91m                                                      By Red Hat
\033[0m"""
    
    def print_logo(self):
        print(self.logo)

    async def delete_channels(self, guild):
        for channel in guild.text_channels:
            await channel.delete()

    async def create_channels_and_send_messages(self, guild, channel_name, num_channels, message, num_messages):
        tasks = []
        for _ in range(num_channels):
            tasks.append(guild.create_text_channel(channel_name))
        channels = await asyncio.gather(*tasks)
        
        tasks = []
        for channel in channels:
            for _ in range(num_messages):
                tasks.append(channel.send(message))
        await asyncio.gather(*tasks)

    def start(self):
        token = input("Inserisci il token del bot: ")
        message = input("Inserisci il messaggio da inviare: ")
        num_messages = int(input("Inserisci il numero di messaggi da inviare: "))
        channel_name = input("Inserisci il nome del canale: ")
        num_channels = int(input("Inserisci il numero di canali: "))
        guild_id = int(input("Inserisci l'ID del server (Guild): "))

        loop = asyncio.get_event_loop()
        loop.run_until_complete(self.setup_bot(token, message, channel_name, num_channels, guild_id, num_messages))

    async def setup_bot(self, token, message, channel_name, num_channels, guild_id, num_messages):
        intents = discord.Intents.default()
        intents.guilds = True
        intents.messages = True

        client = discord.Client(intents=intents)

        @client.event
        async def on_ready():
            print(f'Logged in as {client.user}')

            guild = client.get_guild(guild_id)
            if guild is None:
                print("ID del server non valido.")
                await client.close()
                return

            await self.delete_channels(guild)
            await self.create_channels_and_send_messages(guild, channel_name, num_channels, message, num_messages)

            await client.close()

        await client.start(token)

def main():
    tool = DiscordAdminTool()
    
    while True:
        tool.print_logo()
        print("\033[92m[1] Start")
        print("[2] Exit\033[0m")
        choice = input("\033[92mScelta: \033[0m")

        if choice == '1':
            tool.start()
        elif choice == '2':
            break
        else:
            print("\033[91mScelta non valida. Riprova.\033[0m")

if __name__ == "__main__":
    main()
