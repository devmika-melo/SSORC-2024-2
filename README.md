<h1 align="center">TRABALHO FINAL SSROC</h1>
<h2 align="center">CONFIGURAÇÕES DE CACHE NO SQUID</h2>

<p align="center">
<b>Repositório Utilizado Somente Para a Aplicação da Atividade Do Projeto Final da Cadeira de Segurança em Sietemas Operacionais e Redes de Computadores!</b>
</p>

<h2> 📌 Objetivo Da Atividade</h2>
O objetivo desta atividade é que você configure o Squid para ajustar o tamanho do cache, configurar a limpeza automática dos arquivos antigos e personalize outros parâmetros de cache para melhorar o desempenho e a eficiência do servidor proxy.

<h2> ✔️ Materiais</h2>

Para a realização dessa atividade, será necessário alguns pré-requisitos:

- Ter Visturalizado de Sistemas Operacionasi VirtualBox
    - [VirtualBox](https://www.virtualbox.org/wiki/Downloads)
- O Sistema Operacaional Linux Debian
    - [debian-12.9.0-amd64-netins](https://cdimage.debian.org/debian-cd/current/amd64/iso-cd/debian-12.9.0-amd64-netinst.iso)

<h2>  ✔️ Métodos</h2>
Apara realização dessa atividade é essencial a instalação de alguns programas na sua máquina Debian.
Abra sua máquina vitualLinux Debian como administrador e instale os programas essenciais:

<h4>📍 Instalação do Proxy Squid</h4>

```bash
sudo apt update && sudo apt install squid -y
```
Após a instalação, verificar status do serviços:

```bash
sudo systemctl status squid
```


<h4> 📍 Instalação da Ferramenta curl</h4>

```bash
sudo apt install curl
```
_ferramenta de linha de comando usada para transferir dados entre um cliente e um servidor usando diversos protocolos, como HTTP, HTTPS, FTP, SCP, LDAP, e muitos outros_

<h4> 📍 Instalação da Ferramenta squidclient</h4>

```bash
sudo apt install squidclient
```

_ferramenta usada para interagir com um servidor Squid Proxy. Ele permite que você envie requisições HTTP através do Squid, obtenha estatísticas e depure sua configuração de proxy._

<h2>📝DESENVOLVIMENTO DA ATIVIDADE</h2>

<h3> 🔹 Passo 1: Configurar Cache.</h3>
<h4>Agora vamos configurar o cache no squid</h4>

**📍 Abra em seu terminal o arquivo do squid:**

```bash
sudo nano /etc/squid/squid.conf
```

**📍Adicione as seguinte linhas ao final do arquivo:**

 _(Para ir ao final do arquivo de forma rápida pressione Ctrl + W e depois Ctrl + V)_

 ```ini
# Configura o diretório onde o Squid armazenará os arquivos em cache.
cache_dir ufs /var/spool/squid 2000 16 256

# Define o tamanho máximo dos arquivos que o Squid pode armazenar no cache.
maximum_object_size 4096 KB

# Para de excluir arquivos automáticamente quando chegar em 90% da sua capacidade.
cache_swap_low 90

# Começa a excluir arquivos antigos automáticamente quando atingir 95% da capacidade total para liberar espaço.
cache_swap_high 95

# Permite que todas as máquinas conectadas ao ip da máquina squid armazenem no cache
acl localnet src 192.168.1.40/24 (coloque o ip da sua máquina)

# Permite conexões HTTP (porta 80) e HTTPS (porta 443), garantindo que os usuários possam navegar normalmente sem bloqueios.
acl SSL_ports port 443
acl Safe_ports port 80
acl Safe_ports port 443
acl CONNECT method CONNECT

# Permite que o método CONNECT seja utilizado somente para a porta SSL configurada anteriormente.
http_access allow CONNECT SSL_ports
http_access allow Safe_ports
```

Verifique se as linhas "http_access allow localnet, http_access allow localhost e http_access deny all" estão descomentadas, caso não estejam, descomente.
_(Para ir direto para as linhas pressione Crtl + W e digite a linha na qual deseja achar)_

Também procure a linha "http_port 3128", se estiver comentada, descomente e modifique para "http_port 0.0.0.0:3128"
_Isso permite a conexão tanto para IPv4 quando para IPv6_

## 📌 Configuração de ACLs no Squid

O Squid usa listas de controle de acesso (ACLs) para gerenciar quais conexões e portas são permitidas. Abaixo está um resumo das principais ACLs configuradas para permitir tráfego HTTP e HTTPS.

### 🔹 **Resumo das ACLs**
| **Linha** | **O que faz?** | **Explicação** |
|-----------|---------------|---------------|
| `acl SSL_ports port 443` | Permite conexões HTTPS na porta 443 | O Squid precisa saber que essa porta é usada para HTTPS. |
| `acl Safe_ports port 80` | Permite conexões HTTP na porta 80 | Sem isso, o Squid bloquearia acesso a sites HTTP. |
| `acl Safe_ports port 443` | Permite conexões HTTPS na porta 443 | Sem isso, o Squid bloquearia acesso a sites HTTPS. |
| `acl CONNECT method CONNECT` | Permite que o Squid trate conexões HTTPS usando o método `CONNECT` | Isso possibilita a criação de túneis seguros para HTTPS. |
     
_Com essas configurações, o Squid permite conexões **HTTP (porta 80)** e **HTTPS (porta 443)**, garantindo que os usuários possam navegar normalmente sem bloqueios._

<h3> 🔹 Passo 2: Reiniciar Squid.</h3>

**📍Para aplicar as mudanças, reiniciar o Squid:**

```bash
sudo systemctl restart squid
```

**📍Verifique se o squid está rodando corretamente:**
```bash
sudo systemctl status squid
```
Se aparecer **active (running)**, significa que está rodando!✅

<h3> 🔹 Passo 3: Configurar o Navegador Para Usar o Proxy.</h3>
Agora, precisamos configurar o navegador para utilizar o Squid.

**Se deseja que sua própria máquina virtual acesse o seu proxy cache, siga os passos abaixo:**

**Firefox**
1. Abra o Firefox dá própria VM.
2. Vá nos 3 pontos `☰`  ⭢ **Settings**, desça até o final da página e encontrará **Network Settings** ⭢ **Settings**.
3. Escolha a opção **Manual proxy confuguration**.
4. No campo **HTTP Porxy**, insira:
    - **Endereço IP:** `127.0.0.1`
_Esse é o IP localhost. envia as requisições do Squid rodando na mesma máquina._
    - **Porta:** `3128`
_Porta padrão do squid para comunicação de proxy._

5. Marque a opção **"Also use this proxy for HTTPS"**
6. Clique em **OK** e reinicie o navegador.

**Agora, nos outros computadores da rede:**

1. Vá para Configurações de Rede do navegador (Firefox, Edge, Chromel, Opera, Etc).
2. Procure a opção de **Configuração de Proxy Manual.**
3. Em HTTP Proxy, coloque o IP da máquina virtual e a porta 3128.

**Exemplo:**

Proxy: `192.168.1.50`

Porta: `3128`

Salve e teste acessando um site.


<h3> 🔹 Passo 4: Testar o Funcionamento do Cache.</h3>
Acesse qualquer site no navegdor da VM, por exemplo:

https://www.youtube.com/

**📍Agora, verifique se o cache está funcionando no Squid:**

```bash
sudo tail -f /var/log/squid/access.log
```

_Exibe as últimas linhas do arquivo de log de acessos do Squid_

Você pode dividir a tela do navegador Firefox com o terminal, sem fechar o cache e ver em tempo real o site sendo armazenado.

<img src="img/pritn log cache.jpg" width="700px;" alt="print cache"/><br>

<h2 id="colab">🤝 Collaboradores</h2>

Colaboradores do Projeto

<table>
  <tr>
    <td align="center">
      <a href="https://github.com/devmika-melo" target="_blank">
        <img src="img/ingrid melo.jpg" width="100px;" alt="Ingrid Melo"/><br>
        <sub>
          <b>Ingrid Melo</b>
        </sub>
      </a>
    </td>
    <td align="center">
      <a href="https://github.com/RafaellVieira" target="_blank">
        <img src="img/rafael vieira.jpg" width="100px;" alt="Rafael Vieira"/><br>
        <sub>
          <b>Rafael Vieira</b>
        </sub>
      </a>
    </td>
  </tr>
</table>
