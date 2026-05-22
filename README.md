# Configuração de Ambiente Python Nativo no Windows (PowerShell)

> **Guia profissional para configuração de um ambiente de programação Python nativo no Windows. Focado em fluxos de trabalho de Engenharia e Ciência de Dados no GAAP, com suporte direto para comunicação COM (Aspen Plus / Aspen Hysys).**

Este guia foi elaborado para cenários onde a utilização do Windows Subsystem for Linux (WSL) não é aplicável. Softwares de simulação de processos, como **Aspen Plus** e **Aspen Hysys**, rodam exclusivamente no Windows e exigem que o Python esteja instalado nativamente no mesmo sistema operacional para que a comunicação através do protocolo COM (Component Object Model) funcione corretamente. 

Em vez do terminal Ubuntu e `zsh`, utilizaremos o **Windows Terminal** em conjunto com o **PowerShell**, adaptando as melhores ferramentas de desenvolvimento (como pyenv e git) para o ecossistema Windows nativo.

---

## 📑 Sumário

1. [Windows Terminal e PowerShell](#1-windows-terminal-e-powershell)
2. [Instalação do Git e GitHub CLI](#2-instalação-do-git-e-github-cli)
3. [Visual Studio Code](#3-visual-studio-code)
4. [Instalação do Pyenv-Win](#4-instalação-do-pyenv-win)
5. [Gerenciamento de Ambientes Virtuais (venv)](#5-gerenciamento-de-ambientes-virtuais-venv)
6. [Comunicação com Aspen e Pacotes Python](#6-comunicação-com-aspen-e-pacotes-python)
7. [Configuração do Jupyter Notebook](#7-configuração-do-jupyter-notebook)

---

## 1. Windows Terminal e PowerShell

O **Windows Terminal** é a ferramenta moderna da Microsoft para gerenciar terminais. Por padrão, utilizaremos o **PowerShell**, que é a linguagem nativa de automação do Windows.

### Instalando o Windows Terminal
Caso você esteja no Windows 10 e não tenha o Windows Terminal:
1. Abra a **Microsoft Store**.
2. Busque por "Windows Terminal" e clique em instalar. (No Windows 11, ele já vem instalado por padrão).

### Configurando Políticas de Execução (Muito Importante)
Por padrão, o Windows bloqueia a execução de scripts no PowerShell por questões de segurança. Como precisaremos rodar scripts para ativar os ambientes virtuais do Python, precisamos alterar essa política de forma segura (permitindo scripts locais).

1. Abra o menu Iniciar, digite **PowerShell**.
2. Clique com o botão direito e selecione **Executar como Administrador**.
3. Rode o seguinte comando:

```powershell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
```
4. Pressione "S" (ou "Y") para confirmar.

5. Feche a janela de administrador.

2. Instalação do Git e GitHub CLI
O controle de versão é essencial. Para instalar ferramentas no Windows de forma rápida pelo terminal, usaremos o gerenciador de pacotes nativo winget.

No seu PowerShell, execute:

```powershell
winget install --id Git.Git -e --source winget
winget install --id GitHub.cli -e --source winget
```
[!TIP]
Se o terminal pedir permissão para aceitar os termos de contrato da Microsoft, digite "Y" e pressione Enter.

Após a instalação, feche o terminal e abra um novo para carregar as variáveis de ambiente.

Autenticação no GitHub via CLI
Rode o comando abaixo e siga as instruções na tela, selecionando HTTPS ou SSH conforme sua preferência, e faça login pelo navegador:

```powershell
gh auth login
```
Configurando seu nome e email no git:

```powershell
git config --global user.name "Seu Nome Completo"
git config --global user.email "seu.email@exemplo.com"
```

3. Visual Studio Code
O VSCode será nosso editor principal.

Instale o VSCode através do terminal usando o winget:

```powershell
winget install -e --id Microsoft.VisualStudioCode
```
Ou baixe diretamente pelo site oficial: https://code.visualstudio.com/download

[!IMPORTANT]
Feche e abra o terminal para garantir que o comando code está funcionando.

Melhorando a Experiência no VSCode
Podemos instalar todas as extensões úteis diretamente pelo PowerShell. Copie e cole todos os comandos de uma vez:

```powershell
code --install-extension ms-python.python
code --install-extension KevinRose.vsc-python-indent
code --install-extension ms-python.vscode-pylance
code --install-extension ms-toolsai.jupyter
code --install-extension emmanuelbeziat.vscode-great-icons
code --install-extension MS-vsliveshare.vsliveshare
code --install-extension alexcvzz.vscode-sqlite
```

4. Instalação do Pyenv-Win
O pyenv é maravilhoso para gerenciar versões do Python, mas a versão que usamos no Linux não funciona nativamente no Windows. Para resolver isso, utilizamos o pyenv-win.

No PowerShell, execute o script de instalação oficial:

```powershell
Invoke-WebRequest -UseBasicParsing -Uri "[https://raw.githubusercontent.com/pyenv-win/pyenv-win/master/pyenv-win/install-pyenv-win.ps1](https://raw.githubusercontent.com/pyenv-win/pyenv-win/master/pyenv-win/install-pyenv-win.ps1)" -OutFile "./install-pyenv-win.ps1"; &"./install-pyenv-win.ps1"
```
Após finalizar, o script já deve ter adicionado as variáveis no seu Windows. Feche o terminal e abra novamente.

Verifique se a instalação funcionou digitando:

```powershell
pyenv --version
```

Instalando o Python via Pyenv-Win
Agora, instalaremos a versão desejada do Python e a definiremos como a versão global do seu Windows:

```powershell
pyenv install 3.12.7
pyenv global 3.12.7
```
Confirme com:

```powershell
python --version
```

5. Gerenciamento de Ambientes Virtuais (venv)
Como não estamos no Linux usando pyenv-virtualenv, a melhor e mais robusta maneira de criar ambientes isolados no Windows nativo é utilizando o módulo embutido venv.

Sempre que iniciar um novo projeto, crie uma pasta para ele e instancie um ambiente virtual.

```powershell
# Cria uma pasta para o projeto e entra nela
mkdir meu_projeto_aspen
cd meu_projeto_aspen

# Cria o ambiente virtual chamado ".venv"
python -m venv .venv

# Ativa o ambiente virtual
.\.venv\Scripts\Activate.ps1
```

[!NOTE]
Quando o ambiente estiver ativo, você verá um (.venv) verde no início do prompt do seu PowerShell. Para sair do ambiente, basta digitar:
```powershell
deactivate
```

Sempre ative o .venv antes de abrir o VSCode (code .) ou de instalar pacotes (pip install).

6. Comunicação com Aspen e Pacotes Python
Para interagir com o Aspen Plus e o Aspen Hysys via Python, precisamos do pacote pywin32, que permite ao Python utilizar a interface COM do Windows.

Com o seu ambiente virtual ativado, crie um arquivo requirements.txt na pasta do seu projeto. Você pode usar a mesma lista base do tutorial anterior, mas precisamos garantir a presença dos pacotes essenciais para Windows:

```Plaintext
numpy
pandas
scipy
matplotlib
seaborn
jupyter
ipykernel
openpyxl
pywin32
```

Instale os pacotes:

```powershell
python -m pip install --upgrade pip
pip install -r requirements.txt
```

🧪 Testando a comunicação com o Aspen
Crie um arquivo test_aspen.py no VSCode e adicione o seguinte trecho:

```Python
import win32com.client as win32

try:
    # Para se conectar a uma simulação do Aspen Hysys (já aberta)
    hysys = win32.Dispatch("HYSYS.Application")
    print("Sucesso: Conectado à API COM do Hysys!")
    
    # Exemplo: Imprimir o nome do caso ativo
    # active_case = hysys.ActiveDocument
    # print(active_case.Title)
except Exception as e:
    print(f"Erro ao conectar ao Hysys: {e}")
```

[!WARNING]
É recomendável abrir o software do Aspen (Plus ou Hysys) antes de rodar o script em Python que aciona a API COM, pois isso evita falhas de inicialização do processo OLE do Windows.

7. Configuração do Jupyter Notebook
Para garantir que o Jupyter reconheça corretamente o seu ambiente virtual (e não use o Python global do sistema):

Com o .venv ativado, instale o kernel para este projeto:

```powershell
python -m ipykernel install --user --name=meu_projeto_aspen
```
Inicie o Jupyter Notebook normalmente:

```powershell
jupyter notebook
```
Na interface web, ao criar um novo notebook, você poderá selecionar o kernel meu_projeto_aspen, garantindo que todas as bibliotecas (incluindo o pywin32 para comunicação com o Aspen) estejam acessíveis ao seu código.

🎉 Conclusão
Seu ambiente Windows Nativo está configurado! Diferente do WSL, este setup opera diretamente na mesma camada do sistema em que os softwares de Engenharia (Aspen) rodam, garantindo a perfeita integração via Windows COM, sem abrir mão das ferramentas profissionais de desenvolvimento de software em Python.

Copyright (c) 2026 GAAP - Grupo de Aplicações Avançadas em Processos.
