import discord
from discord.ext import commands, tasks
import requests
from bs4 import BeautifulSoup

# Configuração do bot
token = "DISCORD TOKEN"
intents = discord.Intents.default()
intents.message_content = True
bot = commands.Bot(command_prefix="!", intents=intents)

# ID do canal onde as vagas serão postadas
channel_id = ID DO CANAL

# Lista de categorias de vagas
categorias_vagas = {
    "Engenharia de Dados": ["Engenheiro de Dados", "Data Engineer", "Big Data Engineer", "ETL Engineer", "Data Architect"],
    "Desenvolvimento": ["Desenvolvedor Python", "Software Engineer", "Backend Developer", "Desenvolvedor Backend"],
    "QA": ["Quality Assurance", "Test Analyst", "QA Engineer", "Test Automation Engineer"]
}

# Criar uma sessão global para autenticação no LinkedIn
session = requests.Session()
login_url = "https://www.linkedin.com/uas/login-submit"

# Insira seu e-mail e senha do LinkedIn
login_payload = {
    "session_key": "email",
    "session_password": "senha",
}

# Headers para evitar bloqueio
headers = {
    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/122.0.0.0 Safari/537.36",
    "Accept-Language": "pt-BR,pt;q=0.9,en-US;q=0.8,en;q=0.7",
    "Referer": "https://www.linkedin.com/login",
}

# Fazer login
login_response = session.post(login_url, headers=headers, data=login_payload)

if "feed" in login_response.url:
    print("✅ Login realizado com sucesso!")
else:
    print("❌ Erro ao fazer login! Verifique o e-mail e senha.")

# Função para buscar vagas no LinkedIn (Apenas as 10 primeiras)
def buscar_vagas(palavras_chave, cidade="São Paulo, Brasil"):
    palavra_chave_query = "%20OR%20".join([p.replace(" ", "%20") for p in palavras_chave])
    vagas = set()

    url = (
        f"https://www.linkedin.com/jobs/search/?keywords={palavra_chave_query}&"
        f"f_TPR=r604800&location=Brasil&geoId=106057199&start=0"
    )

    print(f"🔗 Buscando vagas em: {url}")
    response = session.get(url, headers=headers)

    if response.status_code != 200:
        print(f"❌ Erro ao acessar o LinkedIn, código: {response.status_code}")
        return ["⚠ Erro ao buscar vagas no LinkedIn."]

    soup = BeautifulSoup(response.text, "html.parser")

    for job in soup.find_all("div", class_="base-card")[:10]:  # Apenas 10 primeiras vagas
        try:
            titulo = job.find("h3", class_="base-search-card__title").text.strip()
            empresa = job.find("h4", class_="base-search-card__subtitle").text.strip()
            localizacao = job.find("span", class_="job-search-card__location").text.strip()
            link = job.find("a", class_="base-card__full-link")["href"]
            descricao = "Descrição não disponível."

            vaga_tuple = (titulo, empresa, localizacao, link, descricao)
            vagas.add(vaga_tuple)  # Evita duplicatas
        except Exception as e:
            print(f"⚠ Erro ao extrair uma vaga: {e}")

    print(f"📌 Total de vagas únicas extraídas: {len(vagas)}")
    return list(vagas)  # Retorna apenas as 10 primeiras vagas

# Função para limpar o canal antes de postar novas vagas, mantendo a mensagem fixa
async def limpar_canal(channel):
    mensagens = [message async for message in channel.history(limit=100)]
    mensagem_fixa = None

    for mensagem in mensagens:
        if mensagem.author == bot.user and "**Bem-vindo ao canal de vagas!**" in mensagem.content:
            mensagem_fixa = mensagem
            break

    await channel.purge()  # Limpa todas as mensagens do canal

    if mensagem_fixa:  # Mantém a mensagem fixa no topo
        await channel.send(mensagem_fixa.content)

# Tarefa automática para postar vagas a cada hora
@tasks.loop(hours=1)
async def postar_vagas():
    channel = bot.get_channel(channel_id)
    if not channel:
        print("❌ Canal não encontrado.")
        return

    await limpar_canal(channel)  # Limpa o canal antes de postar novas vagas

    # Envia mensagem fixa no canal
    await channel.send(
        "📢 **Bem-vindo ao canal de vagas!**\n"
        "🔎 Aqui postamos as **10 vagas mais recentes** de cada categoria a cada hora.\n"
        "🔄 Para atualizar manualmente, digite `!refresh`.\n"
        "📌 Categorias disponíveis: **Engenharia de Dados, Desenvolvimento, QA**"
    )

    for categoria, palavras_chave in categorias_vagas.items():
        vagas = buscar_vagas(palavras_chave)

        if not vagas or not isinstance(vagas[0], tuple):
            await channel.send(f"⚠ Nenhuma vaga encontrada para **{categoria}** no momento.")
            continue

        embed = discord.Embed(title=f"📢 Vagas de {categoria}", color=0x00ff00)

        for titulo, empresa, localizacao, link, descricao in vagas:
            embed.add_field(
                name=f"🎯 {titulo}",
                value=f"🏢 {empresa} 📍 {localizacao}\n🔗 [Acesse a vaga]({link})",
                inline=False
            )

        await channel.send(embed=embed)  # Enviar embed com as vagas da categoria

# Evento para quando o bot estiver pronto
@bot.event
async def on_ready():
    print(f'Logado como {bot.user}')
    print(f'Prefixo do bot: {bot.command_prefix}')
    print(f'Comandos disponíveis: {[command.name for command in bot.commands]}')
    if not postar_vagas.is_running():
        postar_vagas.start()

# Comando manual para atualizar as vagas e limpar o canal
@bot.command(name="refresh")
async def refresh_vagas(ctx):
    channel = ctx.channel
    await limpar_canal(channel)  # Agora também limpa o canal ao rodar !refresh
    await postar_vagas()
    await ctx.send("✅ Canal atualizado com novas vagas!")

# Comando de teste para ver se o bot responde
@bot.command()
async def ping(ctx):
    await ctx.send("🏓 Pong!")

bot.run(token)
