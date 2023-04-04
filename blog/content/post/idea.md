+++
title = "Intellij IDEA"
description = "Under the hood"
date = 2023-04-04T09:18:18-03:00
tags = ["Java", "IDE", "Intellij IDEA"]
draft = false
weight = 8
+++

## Introdução

### Pré-requisitos

O pré-requisito para rodar bem o Intellij IDEA é ter uma máquina com pelo menos 8GB de RAM e um processador com 4 núcleos. O ideal é ter 16GB de RAM e um processador com 8 núcleos segundo o **[próprio site da JetBrains](https://www.jetbrains.com/help/idea/prerequisites.html#min_requirements)**.

## Under the hood 

### O que ocorre quando você inicia o Intellij IDEA? 

Quando você inicia o IntelliJ IDEA, uma série de processos ocorre em segundo plano para configurar o ambiente de desenvolvimento e prepará-lo para o uso. Vamos detalhar esses processos e entender o papel de cada componente no funcionamento do IntelliJ IDEA.

### Inicialização:

Quando você executa o IntelliJ IDEA, o aplicativo carrega sua interface gráfica e verifica a existência de atualizações e plugins. Dependendo da configuração do seu sistema, você também pode ser solicitado a selecionar um JDK (Java Development Kit) para o projeto. A JVM (Java Virtual Machine) é iniciada para executar o aplicativo e alocar a memória necessária. Se você estiver criando um novo projeto, o IntelliJ IDEA solicitará que você selecione um diretório para armazenar os arquivos do projeto. Dentro deste diretório, várias pastas serão criadas para gerenciar diferentes aspectos do projeto:

```bash
<project_name>
├── .idea
│   ├── modules.xml
│   ├── workspace.xml
│   ├── misc.xml
│   └── <module_name>.iml
├── src
│   └── <package_name>
│       └── <source_file>.java
└── out
    └── production
        └── <project_name>
            └── <compiled_class>.class
```
        
* **.idea**: Esta pasta contém arquivos de configuração específicos do IntelliJ IDEA, como informações do projeto e do módulo, configurações do workspace e arquivos de cache. Essa pasta geralmente não é compartilhada entre desenvolvedores. Esta pasta contém arquivos de configuração específicos do IntelliJ IDEA e é criada automaticamente para cada projeto. Ela inclui:
  * **modules.xml**: Define os módulos do projeto e suas dependências.
  * **workspace.xml**: Armazena as configurações do ambiente de desenvolvimento, incluindo configurações de execução e depuração.
  * **misc.xml**: Contém configurações diversas, como o JDK do projeto.
  * **.iml**: Arquivos de configuração de módulo específicos do IntelliJ que contêm informações sobre as dependências do módulo, o SDK e o compilador.
* **src**: Esta pasta contém o código-fonte do seu projeto, incluindo arquivos Java, arquivos de recursos e outros arquivos relacionados.
* **out ou target**: Esta pasta armazena os arquivos compilados gerados pelo IntelliJ IDEA ou por ferramentas de compilação externas, como o Maven ou o Gradle. Os arquivos nesta pasta são gerados automaticamente e geralmente não são incluídos no controle de versão.

### Cache e Índices:

O IntelliJ IDEA utiliza cache e índices para acelerar a navegação, pesquisa e compilação de código. Essas informações são armazenadas no diretório de cache do sistema, que geralmente está localizado em `~/.cache/JetBrains/IntelliJIdea<versão>` no Linux, `%LOCALAPPDATA%\JetBrains\IntelliJIdea<versão>` no Windows e `~/Library/Caches/JetBrains/IntelliJIdea<versão>` no macOS. Os índices são usados para melhorar o desempenho de pesquisas e inspeções de código, enquanto o cache armazena dados temporários que podem ser reutilizados para acelerar a inicialização e a compilação do projeto. O IntelliJ IDEA usa o padrão de projeto Maven por padrão para organizar a estrutura do diretório do projeto e gerenciar dependências. O padrão Maven é amplamente utilizado na comunidade Java e facilita a colaboração entre desenvolvedores e a integração com outras ferramentas de desenvolvimento. No entanto, o IntelliJ IDEA também suporta outros padrões de projeto, como o Gradle. Depois de criar ou abrir um projeto, o IntelliJ IDEA configura o ambiente de desenvolvimento, incluindo a criação de uma configuração de execução e depuração, a vinculação do JDK e a configuração de outras ferramentas e plugins específicos do projeto. Essas configurações são armazenadas no arquivo `.idea/workspace.xml`.

### Customizando o IntelliJ IDEA

A pasta de cache do IntelliJ IDEA cresce principalmente devido ao armazenamento temporário de dados e arquivos gerados durante o uso do IDE. Esses dados incluem informações de índice, histórico de versões, arquivos compilados, configurações de execução e outros dados temporários. O cache é usado para melhorar o desempenho e a eficiência do IntelliJ IDEA, permitindo acesso rápido a informações frequentemente usadas e evitando a necessidade de regerar dados. No entanto, com o uso contínuo, a pasta de cache pode continuar a crescer e ocupar espaço significativo em disco. Isso acontece porque, em alguns casos, os arquivos temporários não são excluídos automaticamente pelo IDE. Além disso, o cache pode incluir dados de projetos que não estão mais em uso ou informações desatualizadas. Para controlar o crescimento da pasta de cache e liberar espaço em disco, você pode seguir estas etapas:

* **Limpar o cache do sistema**: No IntelliJ IDEA, vá para **File > Invalidate Caches / Restart**. Isso abrirá uma janela de diálogo onde você pode selecionar **Invalidate and Restart**. Isso limpará o cache do sistema e reiniciará o IntelliJ IDEA.
* **Excluir arquivos temporários manualmente**: Navegue até a pasta de cache do IntelliJ IDEA (geralmente localizada em `C:\Users\<username>\.IntelliJIdea<version>\system\` no Windows ou `~/Library/Caches/JetBrains/IntelliJIdea<version>/` no macOS) e exclua os arquivos e pastas que não são mais necessários. Certifique-se de fechar o IntelliJ IDEA antes de fazer isso para evitar possíveis problemas.
* **Configure o tamanho máximo do cache**: Você pode limitar o tamanho do cache ajustando as configurações do IntelliJ IDEA. Para fazer isso, vá para **Help > Edit Custom Properties** e adicione a seguinte linha: `idea.max.content.cache.size=<tamanho_em_MB>`. Salve o arquivo e reinicie o IntelliJ IDEA.

Ou se preferir, execute os scripts abaixo de acordo com cada OS:

### Linux

```bash
#!/bin/bash

# Remove o cache do IntelliJ IDEA
rm -rf ~/.cache/JetBrains/IntelliJIdea*/system/caches

# Adicionando o parametro idea.max.content.cache.size no custom properties
# Lembre-se de usar o valor que lhe for mais conveniente
echo "idea.max.content.cache.size=1024" >> ~/.IntelliJIdea*/config/options/custom.properties
```

### Windows

```powershell
# Remove o cache do IntelliJ IDEA
Remove-Item -Path "$env:LOCALAPPDATA\JetBrains\IntelliJIdea*\system\caches" -Recurse -Force
# Adicionando o parametro idea.max.content.cache.size no custom properties
# Lembre-se de usar o valor que lhe for mais conveniente
Add-Content -Path "$env:LOCALAPPDATA\JetBrains\IntelliJIdea*\config\options\custom.properties" -Value "idea.max.content.cache.size=1024"
```

### macOS

```bash
#!/bin/bash

# Remove o cache do IntelliJ IDEA
rm -rf ~/Library/Caches/JetBrains/IntelliJIdea*/system/caches

# Adicionando o parametro idea.max.content.cache.size no custom properties
# Lembre-se de usar o valor que lhe for mais conveniente
echo "idea.max.content.cache.size=1024" >> ~/Library/Preferences/JetBrains/IntelliJIdea*/options/custom.properties
```

O arquivo `idea.properties` que se encontra no caminho  **Help > Edit Custom Properties**, permite personalizar várias configurações do IntelliJ IDEA. Aqui estão algumas configurações úteis que você pode adicionar ou modificar neste arquivo:

* **idea.max.content.cache.size**: Define o tamanho máximo do cache do IntelliJ IDEA. O valor padrão é 1024 MB.
* **idea.max.intellisense.filesize**: Define o tamanho máximo do arquivo para o qual o IntelliSense é aplicado. O valor padrão é 100 MB.
* **idea.max.intellisense.filesize.java**: Define o tamanho máximo do arquivo para o qual o IntelliSense é aplicado para arquivos Java. O valor padrão é 100 MB.
* **idea.max.intellisense.filesize.xml**: Define o tamanho máximo do arquivo para o qual o IntelliSense é aplicado para arquivos XML. O valor padrão é 100 MB.

> **Nota**: Perceba que você pode alterar o valor do intellisense para cada linguagem, como Java, XML, etc.

* **idea.system.path=<caminho_do_diretório_do_sistema>**: Define o caminho do diretório do sistema do IntelliJ IDEA. O valor padrão é `~/.IntelliJIdea<version>/system`.
* **idea.config.path=<caminho_do_diretório_de_configuração>**: Define o caminho do diretório de configuração do IntelliJ IDEA. O valor padrão é `~/.IntelliJIdea<version>/config`.
* **idea.heap.size.initial=<tamanho_inicial_em_MB>**: Define o tamanho inicial da memória heap do IntelliJ IDEA. O valor padrão é 256 MB.
* **idea.heap.size.maximum=<tamanho_máximo_em_MB>**: Define o tamanho máximo da memória heap do IntelliJ IDEA. O valor padrão é 750 MB.
* **idea.max.intellisense.filesize=<tamanho_em_KB>**: Define o tamanho máximo do arquivo para o qual o IntelliSense é aplicado. O valor padrão é 100 MB.
* **idea.plugins.path=<caminho_do_diretório_de_plugins>**: 
* **idea.save.files.automatically=false**: Desativa o salvamento automático de arquivos.
* **idea.cycle.buffer.size=<tamanho_em_KB>**: Define o tamanho do buffer de ciclo de vida do IntelliJ IDEA. O valor padrão é 100 MB.
* **idea.popup.weight=0.6**: Define a largura máxima dos pop-ups do IntelliJ IDEA como uma fração da largura da tela.
* **idea.smooth.scrolling=true**: Ativa o rolagem suave do IntelliJ IDEA.
* **idea.max.recent.projects=<número_de_projetos>**: Define o número máximo de projetos recentes que o IntelliJ IDEA deve exibir no menu **Arquivo > Abrir recentemente**.
* **idea.use.default.antialiasing.in.editor=true**: Ativa o antialiasing padrão no editor do IntelliJ IDEA.
* **disable.process.cache=true**: Desativa o cache de processos do IntelliJ IDEA. Isso pode ser útil se você estiver tendo problemas com o IntelliSense e queda de desempenho.

Se desejar agilizar o processo de customização do IntelliJ IDEA, você poderá editar as sugestões de scripts acima de acordo com seu SO e necessidade. Dessa forma, você poderá criar um script para cada configuração que desejar e executá-lo sempre que precisar caso mude de ambiente, por exemplo.

## Instalando plugins no IntelliJ IDEA

Você sabia que é possível instalar plugins no IntelliJ IDEA via linha de comando? Para isso, basta executar o comando abaixo:

```bash
$ ./idea.sh install-plugin <plugin_name>
``` 

Exemplo:

```bash
$ # instalando o plugin github-copilot
$ ./idea.sh install-plugin github-copilot
```

Agora podemos explorar isso de uma forma mais divertida. Vamos criar um script que instala uma lista de plugins no IntelliJ IDEA. Para isso, crie um arquivo chamado `install-plugins.sh` e adicione o seguinte conteúdo:

```bash
#!/bin/bash

# Lista de plugins que serão instalados

PLUGINS=(
    "github-copilot"
    "gitignore"
    "lombok"
    "material-theme-ui"
    "sonarlint"
    "sonarqube-community"
    "string-manipulation"
    "yaml"
)

# Instalando os plugins
for plugin in "${PLUGINS[@]}"; do
    ./idea.sh install-plugin $plugin
done
```

Agora edite o arquivo `idea.properties` e adicione o seguinte conteúdo:

```bash
idea.plugins.path=~/plugins
```

Ou se preferir, você pode usar o diretório padrão que fica em `~/.IntelliJIdea<version>/config/plugins`. Agora execute o script `install-plugins.sh` e veja os plugins sendo instalados no IntelliJ IDEA:

```bash
$ ./install-plugins.sh
```

Bastante prático, não é mesmo? Agora você pode criar um script para cada configuração que desejar e executá-lo sempre que precisar caso mude de ambiente.

## Customizando a JVM do IntelliJ IDEA

O IntelliJ IDEA pode consumir bastante recursos no geral. Por isso, é importante que você configure a JVM corretamente para que o software funcione de forma satisfatória. Para isso, você pode ir em Help > Edit Custom VM Options que irá abrir o arquivo `idea64.exe.vmoptions` ou `idea64.vmoptions` (dependendo do seu sistema operacional) no diretório de instalação do IntelliJ IDEA. Neste arquivo, você pode adicionar as seguintes configurações:

* **-Xms1024m**: Define o tamanho inicial da memória heap do IntelliJ IDEA. O valor padrão é 256 MB.
* **-Xmx2048m**: Define o tamanho máximo da memória heap do IntelliJ IDEA. O valor padrão é 750 MB.
* **-XX:ReservedCodeCacheSize=240m**: Define o tamanho do cache de código reservado do IntelliJ IDEA. O valor padrão é 240 MB.
* **-XX:+UseConcMarkSweepGC**: Ativa o coletor de lixo do tipo CMS (Concurrent Mark Sweep) do IntelliJ IDEA.
* **-XX:SoftRefLRUPolicyMSPerMB=50**: Define a política de referência suave do IntelliJ IDEA.
* **-XX:+Use<GC>**: - Define o coletor de lixo a ser usado (G1, Parallel, ConcMarkSweep, etc.)
* **-XX:MaxGCPauseMillis=<value>**: - Define o tempo máximo de pausa para a coleta de lixo
* **-XX:ParallelGCThreads=<value>**: - Define o número de threads usadas pelo coletor de lixo paralelo
* **-XX:NewSize=128m**: Define o tamanho inicial da memória heap do IntelliJ IDEA
* **-XX:MaxNewSize=256m**: Define o tamanho máximo da memória heap do IntelliJ IDEA

Para mais informações, acesse a documentação oficial do IntelliJ IDEA em **[https://www.jetbrains.com/help/idea/tuning-the-ide.html](https://www.jetbrains.com/help/idea/tuning-the-ide.html)**.







