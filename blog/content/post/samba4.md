+++
title = "Ops - Samba 4"
description = "Um novo horizonte"
date = 2020-04-19T17:31:45-03:00
tags = ["linux,samba,fedora"]
draft = false
weight = 4
author = "Vitor Lobo Ramos"
+++

> **Esta é uma documentação não oficial** do Samba 4 que fora elaborada através de um estudo pessoal e experiências práticas que tenho tido com o Samba em ambiente de produção. Se houver por minha parte alguma informação errada, por favor, entre em contato para que eu possa corrigir através do meu e-mail (**lobocode@gmail.com**), ou me mande um **pull request** no github. As referências usadas para o estudo além da experiência prática, estarão no rodapé da página. Documentação em **constante atualização**.

### Capítulo 1 - História

* **[Introdução](#introducao)**
* **[A origem do nome samba](#a-origem-do-nome-samba)**
* **[Histórico de versões](#historico-de-versoes)**

### Capítulo 2 - Conceitos iniciais

* **[Conceitos iniciais](#conceitos-iniciais)**
* **[Compartilhamento de arquivos](#compartilhamento-de-arquivos)**
* **[Protocolos SMB e CIFS](#protocolos-smb-e-cifs)**
* **[Estrutura do Active Directory](#estrutura-do-active-directory)**
* **[Protocolos do Active Directory](#protocolos-do-active-directory)**

### Capítulo 3 - Samba como AD

* **[Codigo fonte do Samba](#codigo-fonte-do-samba)**
* **[Instalando o Samba](#instalando-o-samba)**
* **[Samba como um controlador de dominio](#samba-como-um-controlador-de-dominio)**
* **[Compartilhamento com o servidor de impressao](#compartilhamento-com-o-servidor-de-impressao)**
* **[Lixeira do samba](#lixeira-do-samba)**
* **[Auditando acessos](#auditando-acessos)**
* **[Rotacionamento de logs](#auditando-acessos)**
* **[Acls para permissões avançadas no Linux](#auditando-acessos)**
* **[Migrando um samba3 PDC para Samba 4 AD](#migrando-um-samba3-pdc-para-samba-4-ad)**
* **[Samba Tool](#samba-tool)**

### Capítulo 4 - Ingressando clientes no domínio Windows

* **[RSAT Ferramenta de administracao remota](#rsat-ferramenta-de-administracao-remota)**
* **[Configurando perfil nomade](#configurando-perfil-nomade)**
* **[Adicionando unidades organizacionais](#adicionando-unidades-organizacionais)**
* **[Implementando Group Policies](#implementando-group-policies)**
* **[Backup e recovery do samba AD](#backup-e-recovery-do-samba-ad)**
* **[Samba como servidor secundario](#samba-como-servidor-secundario )**
* **[Testes de replicacao de diretorios](#testes-de-replicacao-de-diretorios)**

### Extras 

* **[Automatizando o provisionamento do samba com Ansible](#)**

### Introdução

Tudo começou em 1983, quando a **[IBM](www.ibm.com)** e a Sytec co-desenvolveram um sistema de rede simples projetado para a construção de pequenas redes locais. O sistema era baseado numa API para comunicação em redes chamada **[NetBIOS](https://pt.wikipedia.org/wiki/NetBIOS)**, ou **Net**work **B**asic **I**nput **O**utput **S**ystem, isto é, **Sistema Básico de Rede de Entrada/Saída**.

Foi incluso no NetBIOS um esquema de endereçamento que usava nomes de 16 bytes para identificar estações de trabalho e aplicações habilitadas para rede. Em seguida, a **[Microsoft](www.microsoft.com)** adicionou recursos para o DOS que permitia o disco I/O ser redirecionado para a interface NetBIOS, **o que permitiu que o espaço em disco fosse compartilhável através da LAN**. O protocolo de compartilhamento de arquivos ficou conhecido como **[SMB](https://pt.wikipedia.org/wiki/Server_Message_Block)**, e agora **[CIFS](http://www.webopedia.com/TERM/C/CIFS.html)**.

Em 1985, o protocolo foi expandido, dando origem ao protocolo **[NetBEUI](https://pt.wikipedia.org/wiki/NetBEUI)**, que foi durante muito tempo o principal protocolo usado em redes locais, antes da popularização do TCP/IP. O **[SMB](https://pt.wikipedia.org/wiki/Server_Message_Block)** (Server Message Block) veio mais tarde, junto com o Windows 3.11. O protocolo SMB faz o compartilhamento de arquivos e impressoras em redes Microsoft, incluindo a navegação na rede, o estabelecimento de conexões e a transferência de dados. Ele utiliza o NetBIOS para a troca de mensagens entre os hosts e inclui uma versão atualizada do protocolo, que roda sobre o TCP/IP.

**[Andrew Tridgell](https://pt.wikipedia.org/wiki/Andrew_Tridgell)**, um australiano que na época era estudante do curso de PhD em Ciências da Computação da **[Universidade Nacional da Austrália](www.anu.edu.au)**, precisava rodar um software chamado **["eXcursion"](http://www.hpl.hp.com/hpjournal/dtj/vol4num1/vol4num1art7.pdf)**, no **[DEC](https://en.wikipedia.org/wiki/Digital_Equipment_Corporation)**, empresa que trabalhava, para interagir com o **[Patchworks](https://en.wikipedia.org/wiki/Pathworks)**, um software de compartilhamento de arquivos. O **[Patchworks](https://en.wikipedia.org/wiki/Pathworks)** era um software proprietário, que utilizava um protocolo obscuro, sobre o qual não existiam muitas informações disponíveis.

![Samba](https://raw.githubusercontent.com/lobocode/lobocode.github.io/master/post/images/samba/andrew.jpg#center)

Como todo bom hacker Andrew_tridgell decidiu estudar o protocolo e assim desenvolver um servidor que pudesse rodar em seu PC. Ele desenvolveu então um **[sniffer](https://pt.wikipedia.org/wiki/Sniffing)** chamado **clockspy** que era capaz de examinar o tráfego da rede e fez engenharia reversa no protocolo SMB e o implementou no Unix. Isso fez com que no servidor Unix aparecesse como um servidor de arquivos Windows em seu PC com DOS. Com isso, ele foi rapidamente capaz de implementar o suporte às principais chamadas e a desenvolver um programa servidor que era capaz de conversar com os clientes rodando o Patchworks. O objetivo desta primeira versão era apenas resolver um problema doméstico: interligar um computador rodando o MS-DOS ao servidor rodando o **[Solaris](https://pt.wikipedia.org/wiki/Solaris)**.

O projeto começou a se tornar popular a partir da versão 1.6.09, que foi a primeira a trazer suporte ao controle de acesso com base nos logins de usuário (assim como o Windows NT), enquanto as versões anteriores suportavam apenas o controle de acesso com base no compartilhamento (assim como no Windows 3.11 e 95), onde a única opção de segurança era usar uma senha de acesso para os compartilhamentos. Para a época, foi um avanço extraordinário visto que tudo o que o samba oferecia, supria os serviços oferecidos pela Microsoft.

No entanto, em 1999 a Microsoft lançou o Active Directory que é implementado pela primeira vez no **[Microsoft Windows 2000 Server Edition](https://pt.wikipedia.org/wiki/Windows_2000)** e com melhorias significativas no Microsoft Windows Server 2003 e 2003 R2 e mais tarde no Windows Server 2008 e 2008 R2.

> Com isto, surgiu o desafio da comunidade que desenvolvia o Samba na época (e que desenvolve até hoje), a criar uma implementação de serviço de diretório usando um protocolo compatível com o **[LDA](https://pt.wikipedia.org/wiki/LDAP)** que armazenasse informações sobre objetos na rede e ao mesmo tempo disponibilizasse essas informações a usuários e administradores da mesma rede. 

O problema é que não havia interesse por parte da Microsoft em compartilhar o **[RFC](https://pt.wikipedia.org/wiki/Request_for_Comments)** (Request for Comments), isto é, o documento que descreve os padrões de cada protocolo de Internet antes de serem considerados de fato um padrão como fora feito com o protocolo LDAP por exemplo. Com isto, a comunidade que desenvolve o Samba não fazia ideia de como o AD funcionava. Basicamente, teriam que pesquisar anos a fio e colocar mais sniffers em rede, fazer engenharia reversa e mais uma vez demorar anos e anos em desenvolvimento para criar uma solução alternativa ao **[AD](https://pt.wikipedia.org/wiki/Active_Directory)**.

Usando destes recursos então a comunidade começa a escrever uma nova versão pós a 3.6 do Samba em 2011. A **[versão 3.6](https://www.samba.org/samba/history/samba-3.6.25.html)** já trabalhava com o protocolo LDAP usando o **[OpenLDAP](https://pt.wikipedia.org/wiki/OpenLDAP)** e o **[Kerberos](https://pt.wikipedia.org/wiki/Kerberos)** em separado para alimentar as funcionalidades do Samba para que pudessem obter funcionalidades próximas ao AD. 

Depois de anos de desenvolvimento, a o que seria supostamente a versão 3.7 do Samba não chega nem a ser lançado e é descontinuado visto que em 2012, momento em que o Samba ainda estava em desenvolvimento, a Microsoft libera o RFC 1823 do LDAP, e o RFC 2307,3062,4533 e a comunidade desiste do desenvolvimento da nova versão do Samba e resolvem reescreve-lo do zero.

Nasce o **[Samba 4.0](https://www.samba.org/samba/history/samba-4.0.0.html)**. O Samba 4 compreende um servidor de diretório LDAP, um servidor de autenticação Kerberos, um servidor DNS dinâmico e implementações de todas as chamadas de procedimento remoto necessários para o Active Directory. Ele fornece tudo que é necessário para servir como um Active Directory Domain Controler compatível com todas as versões de clientes Microsoft Windows atualmente suportados pela Microsoft, incluindo o Windows 8. O Samba agora permite implementar mecanismos de compartilhamento de arquivos e impressoras, sendo assim possível o acesso a recursos do Windows a partir do GNU/Linux.

![Samba](https://raw.githubusercontent.com/lobocode/lobocode.github.io/master/post/images/samba/samba4.png#center)

Possibilidades com Samba4:

* Criar um AD Completo
* Criar um controlador de domínio Principal
* Criar um Controlador de domínio somente leitura (RODC)
* Criar um Controlador de domínio Adicional
* Pode ser administrado usando interface Gráfica do próprio Windows , como Usuários e computadores do Active Directory.
* Posso Migrar de forma fácil de AD Windows para um AD Linux e vice versa.
* É possível trabalhar com perfil móvel
* Trabalhar com Pasta Base
* Lixeira de Servidor de Arquivos (Tipo copia de Sombra)
* Auditoria de Acesso
* Trabalhar com permissões como a do Windows
* **Trabalhar com GPO**
* Fazer replicação de Servidores  (TIPO DFS)
* Trabalhar com dados em camadas
* Triagem de Arquivos  (Proibir gravação de arquivos pela extensão)
* Não precisa de CALs de acesso para as estações
* Posso fazer o SAMBA 4 trabalhar como controlador de domínio adicional do Windows server e vice versa.
* Já vem com DNS , kerberos, LDAP integrado.
* Posso fazer a integração do SAMBA 4 com o proxy Squid, pfSense e etc
* Adicionar usuários e configurações via linha de comando de maneira muito mais interativa e rápida.

---

### A origem do nome Samba

> O nome "Samba" surgiu a partir de uma simples busca dentro do dicionário **[Ispell](https://en.wikipedia.org/wiki/Ispell)** por palavras que possuíssem as letras S, M e B, de "Server Message Blocks", posicionadas nessa ordem. A busca retornou apenas as palavras "salmonberry", "samba", "sawtimber" e "scramble", de forma que a escolha do nome acabou sendo óbvia.

O comando que foi usado para consultar o dicionário Ispell no sistema:
```bash
sudo grep -i '^s.*m.*b' /usr/share/dict/words
```

---

### Historico de Versoes

A primeira versão a usar o nome Samba, foi a 1.6.05, que foi a sucessora imediata do "smbserver 1.6.4". O projeto anteriormente fora apresentado por **[Andrew Tridgell](https://pt.wikipedia.org/wiki/Andrew_Tridgell)** com o nome de **[nbserver 0.5](http://ftp.samba.org/ftp/samba/old-versions/server-05.tar.Z)** que foi o início do projeto que veio a tornar-se Samba mais tarde. Ao todo se considerar-mos os patches, o Samba contém mais de 50 versões até chegar a versão atual. Sem os patches e\ou versões no estilo **[CVS](https://pt.wikipedia.org/wiki/CVS)**, o Samba tem exatamente 4 versões desde a 1.x.x até a recente 4.x.x que podem ser vistos no **[histórico de versões do samba](https://www.samba.org/samba/history/)**. O registro da primeira versão do que seria o samba hoje, foi anunciado por Andrew na [Usenet](https://en.wikipedia.org/wiki/Usenet) (hoje arquivada em no Google Groups) e ainda pode ser visto **[aqui](https://groups.google.com/forum/#!msg/vmsnet.networks.desktop.pathworks/EmyZEGd5ZRA/nJtBfp6ak30J)**.

O importante aqui é destacar as mudanças mais significativas entre as versões até chegar no Samba4. Por exemplo: 

* **1.6.09** Foi a primeira versão a apoiar a segurança a nivel de usuário (o que fez do Samba um concorrente muito mais sério para o uso corporativo na época). 
* **1.7** Suportava a desconfiguração de nomes de arquivos para clientes mais antigos, bem como também foi a primeira versão a ter um FAQ. 
* **1.8** Esta versão foi a primeira a adicionar suporte primitivo para línguas estrangeiras.
* **1.9** Suporte de navegação mais aprimorado.
* **1.9.16** Foi quando foi adotado o **[CVS](https://pt.wikipedia.org/wiki/CVS)** o que permitiu que o Samba fosse desenvolvido mais rapidamente.
* **2.0** O Samba se tornou ainda mais popular e acessível aos usuários finais. Neste mesmo ano 1999, fora lançado o **[primeiro livro sobre o Samba](https://www.samba.org/samba/docs/using_samba/ch00.html)** que hoje faz parte da documentação oficial do mesmo.
* **2.2** Esta se torna a primeira versão estável do Samba. Nesta versão, já é possível criar um domínio primário para **[NT4](https://en.wikipedia.org/wiki/Windows_NT_4.0)**, suporta Windows 2000 e é melhorada a parte de infra-Estrutura para impressão **(graças ao apoio da HP)** e comunidade.
* **3.0** O principal diferencial da Samba 3.0 é que ele inclui suporte para autenticação Kerberos 5 e LDAP, que são obrigados a agir como clientes em um domínio do Active Directory. Outra característica que apareceu no Samba 3.0 é o suporte a Unicode, o que simplifica muito apoio às línguas internacionais. 

> A diferença é que os serviços LDAP, Kerberos e outros, não estão integrados ao Samba nesta versão como está na 4.Segundo Bartlett (2005, p. 57), o SAMBA 3.0 possuía a capacidade de entrar como usuário em uma rede operando com Active Directory. Esta funcionalidade foi o início dos trabalhos com Active Directory, que vieram a tona de forma mais convincente no SAMBA 4.

* **4.0** Esta versão introduz o aguardado suporte para a tecnologia Microsoft Active Directory através da implementação de uma combinação de um servidor de diretório LDAP, um servidor de autenticação Heimdal Kerberos, um serviço DNS dinâmico (através do seu próprio servidor DNS ou plugin BIND) e todo o procedimento de chamadas remotas necessárias para cumprir a função de um controlador de domínio do Active Directory (Active Directory Domain Controller) de todas as versões suportadas do Windows, incluindo o Windows 8.


O Samba 4 fornece políticas de grupo (**G**roup **P**olicies **O**bject), perfis móveis (Roaming Profiles) e outros recursos para administrar sistemas em um domínio do Windows, bem como integração com servidores Exchange e alternativas compatíveis em código aberto. O suporte do Samba ao Active Directory é completamente transparente para os clientes, o que significa que um controlador de domínio baseado em Samba pode ser integrado a domínios do Active Directory existentes. 

Scripts de atualização são fornecidos pelos desenvolvedores do Samba para ajudar com uma migração “suave” de um recurso de controlador de domínio Windows NT no Samba 3.x.

Os recursos de compatibilidade do Active Directory no Samba 4 foram criados com a ajuda da documentação oficial e testes de interoperabilidade da Microsoft. “Estamos satisfeitos que os laboratórios de documentação e de interoperabilidade fornecidos pela Microsoft tenham sido fundamentais no desenvolvimento do Samba 4.0 Active Directory. A Microsoft está empenhada em apoiar a interoperabilidade entre as plataformas”, disse **[Thomas Pfenning](https://www.linkedin.com/pub/thomas-pfenning/a/153/163)**, diretor de desenvolvimento do Windows Server. 

--- 

### Conceitos iniciais

Das muitas tecnologias de redes, uma das mais conhecidas para redes de computadores, especialmente aquelas que usam de forma homogênea o sistema operacional Windows, é o **[Active Directory](https://pt.wikipedia.org/wiki/Active_Directory)**. Por questões de praticidade e segurança, muitas redes de computadores que rodam o Windows, são configuradas com AD, mas isso originalmente implicava na utilização única e exclusiva de sistemas operacionais em toda a rede envolvida ou pelo menos no servidor cujas licenças sempre foram muito mais onerosas do que as versões de Windows para estações de trabalho. O SAMBA sempre buscou a interoperabilidade na comunicação entre sistemas heterogêneos.

Segundo a Microsoft, o Active Directory (AD) é um serviço de diretório desenvolvido pela empresa para o domínio das redes Windows e está incluído na maioria dos sistemas operacionais Windows Server como um conjunto de processos e serviços. Esta tecnologia permite aos usuários de uma rede de computadores trabalharem usando entradas personalizadas de usuário e senha, não importando se o usuário decidir usar outro computador, suas configurações e arquivos pessoais poderão ser acessados, desde que tais máquinas estejam dentro de um mesmo domínio controlado por um servidor.

AD é um controlador de domínio que autentica e autoriza todos os usuários e computadores em uma rede de tipo domínio do Windows, atribui e aplica as políticas de segurança para todos os computadores autenticados, como instalação e atualizações de software. O AD é usado em empresas com redes Windows, nas quais os administradores de redes escolheram utilizar este recurso como forma de centralizar a administração dos recursos, bem como da segurança.

Como facilidade, ele armazena centralmente os dados de usuários e computadores, permitindo que os usuários tenham apenas uma senha para acessar todos os recursos disponíveis na rede. O diretório também pode ser utilizado para compartilhamento de arquivos, impressoras, gerenciamento de atualizações de estações de trabalho através do **WSUS (Windows Server Update Services)**, tudo em um console simples e gerenciável. 

Quando um usuário faz logon em um computador que faz parte de um domínio de rede, o AD verifica a senha apresentada e determina se o usuário é um administrador de
sistema ou um utilizador normal. Uma instância do AD consiste em um banco de dados e o correspondente código executável responsável pelo atendimento de pedidos e a manutenção do banco de dados. A parte executável é chamada de Directory System Agent (DSA), que é uma coleção de serviços do Windows e processos que são executados no Windows 2000 e versões posteriores.

Os objetos no bancos de dados do AD podem ser acessados através do protocolo LDAP, o ADSI (interface dos COMs (Component Object Model) ) e dos serviços da APIs de
mensagens e o Gerenciador de Contas de Segurança. 

No Active Directory são encontrados objetos, florestas, arvores, domínios, partiçoes, esquemas e unidades organizacionais (similares a pastas). Uma estrutura de AD é um arranjo de informações sobre objetos. Os objetos se dividem em duas categorias principais:

* Recursos: Hardware em uma rede ex.: Impressoras
* Entidades de segurança: Entidades puramente lógicas ex.: Contas de usuário ou grupos.

Para cada entidade de segurança são atribuídos security identifiers (SIDs) únicos. Cada objeto representa uma única entidade: um usuário, um computador, uma impressora ou um grupo e seus atributos, e certos objetos podem conter outros objetos. 

Dentro de uma implantação, os objetos são agrupados em domínios. Os objetos para um único domínio são armazenados em uma única base de dados (que podem ser replicados).
Os domínios são identificados pelo seu o namespace (nome no DNS). Um domínio é definido como um grupo lógico de objetos de rede (usuários, dispositivos) que compartilham o mesmo banco de dados AD.

Uma árvore é uma coleção de um ou mais domínios, e árvores de domínio em um namespace contínuo, vinculados em uma hierarquia de confiança transitiva. Uma floresta é uma coleção de árvores que compartilham um catálogo global comum, esquemas de diretório, estruturas lógicas e a configuração do diretório. A floresta representa o limite de segurança dentro do qual os usuários, computadores, grupos e outros objetos são acessíveis. 

O banco de dados do AD está organizado em partições, cada um armazenando tipos de objetos específicos e seguindo um padrão de replicação. A partição schema contém a definição de classes de objetos e atributos dentro da Floresta. A partição Configuration contém informações sobre a estrutura física e de configuração da floresta (como a topologia física). Ambos replicam para todos os domínios da floresta. A partição do domínio detém todos os objetos criados nesse domínio e replica somente dentro de seu próprio domínio. 

Os objetos mantidos dentro de um domínio podem ser agrupados em Organizational Units (OUs). OUs é o nível recomendado de a aplicação de políticas de grupo, que são objetos do AD chamados formalmente **Group Policy Objects (GPOs)**, embora as políticas também podem ser aplicadas a domínios ou sites. A UO é o nível em que os poderes administrativos são comumente delegados, mas a delegação pode ser executada com os objetos individuais ou com os atributos também.

![Samba](https://raw.githubusercontent.com/lobocode/lobocode.github.io/master/post/images/samba/gpo.gif#center)

Não é possível, criar contas de usuários com um nome de usuário idêntico em OUs separadas. Isto é assim porque o atributo de objeto de usuário (sAMAccountName), deve ser exclusivo dentro do domínio. Dois usuários em diferentes OUs podem ter o mesmo Common Name (o nome sob o qual eles são armazenados no próprio diretório).

A razão para este problema de nomes duplicados através de diretórios hierárquicos, é que a Microsoft baseia-se nos princípios do NetBIOS, que é um método de gerenciamento de objetos de redes de arquivos simples, que vem desde o Windows NT 3.1 e MS-DOS LAN Manager. Permitir a duplicação de nomes de objetos no diretório, ou remover completamente o uso de nomes de NetBIOS, impediria a compatibilidade com softwares e equipamentos legados.

**O Volume de Sistema (SYSVOL)** que é um diretório compartilhado que armazena a cópia do servidor de arquivos de domínio público, que deve ser compartilhado para acesso e replicação em domínios comuns. Ela contém:

* O Logon de rede compartilhado. Hospedam scripts de logon e objetos de diretiva
dos computadores clientes.
* Scripts de logon de usuário, para os domínios em que o administrador usa as
opções “Usuários e Computadores” do Active Directory.
* Diretivas de grupo do Windows.
* Os arquivos de transferências, das pastas e arquivos que devem estar disponíveis e
sincronizados entre os controladores dos domínios de serviço de replicação (FRS).
* Junções de sistema de arquivo. Segundo a Microsoft, o recomendável é instalar esses 3 em locais distintos, preferencialmente em HDs separados, para facilitar a recuperação do sistema em caso de falha de hardware. Se a floresta está no nível de funcionalidade do Windows Server 2008 ou Windows Server 2008 R2, o sistema de replicação de SYSVOL deixa de ser o NT File Replication. 
* Service (NTFRS) e passa a ser o Distributed File System Replication (DFSR), o que termina criando uma dependência de uma versão específica de sistema operacional Windows.

![Samba](https://raw.githubusercontent.com/lobocode/lobocode.github.io/master/post/images/samba/sysvol.gif#center)

Depois de 10 anos de trabalho e 6 anos da liberação da última versão principal, os desenvolvedores do Samba anunciaram o lançamento do Samba 4.0, a versão mais recente da implementação em software livre do protocolo Server Message Block (SMB).  Outras características do Samba 4 incluem a primeira implementação em software livre da versão 2.1 do protocolo SMB de compartilhamento de arquivos da Microsoft e uma implementação inicial do SMB3 que será desenvolvido em versões futuras da ramificação do Samba 4. 

Por padrão, o servidor smbd do Samba 3 é utilizado para todos os serviço de arquivos. Para casos de uso como um controlador de domínio do Active Directory, o servidor de arquivos NTVFS é fornecido da forma como foi utilizado nos primeiros betas do Samba 4 e é mais recomendável para esta tarefa. A integração segura e incorporada ao NTP fornece **[timestamps](https://pt.wikipedia.org/wiki/Marca_temporal)** mais precisos para clientes Windows.

---

### Compartilhamento de arquivos

Uma das características mais poderosas do Samba é a sua capacidade de compartilhar arquivos e pastas entre o servidor e o cliente. De forma simplificada, para compartilharmos uma pasta e\ou arquivo no Samba4, basta criar-mos alguns parâmetros simples no smb.conf e\ou usar algum client gráfico, ambiente facilitador para tal. No entanto, como este artigo é focado em administradores, logo, **faremos tudo por linha de comando**:


Crie uma pasta qualquer
```bash
mkdir ~/pasta
```

Abra o arquivo de configuração smb.conf, e crie as seguintes linhas:

```bash
# Crie as seguintes linhas:
[PASTA]
comment = Pasta teste
path = ~/pasta
read only = No
```

Por fim, reinicie as configurações do samba
```bash
sudo smbcontrol all reload-config
```

Neste momento, já podemos acessar o diretório usando o Microsoft Windows em nosso domínio (por exemplo, como administrador) e configurar todas as permissões que quisermos. Além disso, usuários e grupos podem diretamente acessar o compartilhamento criado. Como Samba 4 é compatível com o Active Directory, não precisa usar qualquer modelo antigo para restringir o acesso do usuário no arquivo smb.conf. Mais a frente nesta documentação, veremos detalhadamente como é feito o compartilhamento entre diretórios, arquivos, impressoras, com permissões e muito mais.

---

### Protocolos SMB e CIFS

O protocolo SMB foi desenvolvido inicialmente por **[Barry Feigenbaum](https://www.linkedin.com/in/barryfeigenbaum)** na IBM, na década de 80. Exigia, nessa época, uma camada de API (Application Programming Interface), denominada Network Basic Input/Output System isto é, o **(NetBIOS)**. O NetBIOS fornecia serviços tais como resolução de nome e navegação de rede que posteriormente foi implementado, ou adotado, pela Microsoft em função da expansão de seus produtos na década de 90 que era focado em rede de computadores.

Na evolução do desenvolvimento do SMB, pela Microsoft, criou-se uma versão chamada CIFS **(Common Internet File System)** para padronizar com a **[Internet Engineering Task Force - IETF](https://pt.wikipedia.org/wiki/Internet_Engineering_Task_Force)** recebendo o nome final de **SMB/CIFS** (tecnicamente um "dialeto" do SMB) em 1996. 
O Windows for WorkGroups foi o primeiro sistema da Microsoft a adotar o SMB/CIFS. Como o protocolo predominante foi o TCP/IP e para a resolução de nomes a Microsoft teve que adotar o DNS (Domain Name System) e com que SMB tivesse que ser executado diretamente sobre o protocolo TCP/IP na modalidade denominada **hosting direto** (utiliza-se TCP e UDP na porta 445). Portanto, em redes Windows, o SMB/CIFS se transformou em um padrão para a manipulação de arquivos entre máquinas.

O SMB, que significa Server Message Block, é um protocolo para compartilhamento de arquivos, impressoras, portas seriais e abstrações de comunicação. O SMB é um protocolo cliente-servidor. O diagrama a baixo, ilustra a maneira pela qual funciona SMB. Os servidores disponibilizam sistemas de arquivos e outros recursos (impressoras, processadores de mensagens, pipes) disponíveis para os clientes da rede. Os computadores clientes podem ter seus próprios discos rígidos, mas eles também querem o acesso aos sistemas de arquivos compartilhados e impressoras nos servidores.

![SMB](https://raw.githubusercontent.com/lobocode/lobocode.github.io/master/post/images/samba/smb1.png#center)

Basicamente o cliente se conecta ao servidor que utiliza o protocolo TCP/IP (na verdade, se conecta ao NetBIOS sobre o TCP/IP, conforme especificado no **[RFC1001](http://tools.ietf.org/html/rfc1001)** e **[RFC1002](http://tools.ietf.org/html/rfc1002)**), NetBEUI ou IPX/SPX. Depois de terem estabelecido uma conexão, o cliente pode, em seguida, enviar comandos (SMBs) para o servidor o que lhe permite abrir arquivos, ler e escrever. É possível fazer todo o tipo de coisa dentro de um servidor de arquivos. No entanto, no caso do SMB, estas coisas são feitas através da rede.

O protocolo SMB embora tenha outras funções associadas a ele, primordialmente tem como funcionalidade o compartilhamento de arquivos. Mas é possível também o compartilhamento de impressoras e definir níveis de segurança e autenticação. Por ser muito usado nos sistemas operacionais da Microsoft a versão do SMB, denominado SMB/CIFS, é um protocolo muito comum em diversos tipos de máquinas e sistemas para o compartilhamento de arquivos. 

> O SMB pela perspectiva do Modelo OSI está na **camada de aplicação** e utiliza nomes de até 15 caracteres para definir endereços de máquina em uma rede. A Microsoft chegou ainda a desenvolver o SMB2 juntamente com o lançamento do Windows Vista. 

Andrew Tridgell utilizando da engenharia reversa em cima do protocolo SMB implementou no sistema operacional Unix e fazendo com que o servidor Unix aparecesse como sendo um servidor de arquivos Windows em seu computador com DOS. O Samba foi viabilizado por meio do protocolo **[NBT](https://pt.wikipedia.org/wiki/NetBIOS)**, de 1987, que emula redes locais NetBIOS sobre redes TCP/IP. O NBNS (mais conhecido tecnicamente por WINS - Windows Internet Name Server, ou ainda NBT) cria praticamente uma lista cruzada de endereços IP e nomes NetBios facilitando dessa forma a comunicação entre máquinas e sistemas distintos.

---

### Estrutura do Active Directory

Um serviço de diretório pode se explicado com várias ilustrações, mas a ilustração que, em minha opinião, mais demonstra o verdadeiro sentido de um serviço de diretório é a figura de uma lista telefônica ou uma agenda pessoal. Em nossa agenda podemos organizar, dias, semanas, meses e até anos, passando por pessoas, nomes, sobrenomes, datas de aniversário, dados importantes dentre outros. 

O serviço de diretório tem exatamente o mesmo sentido, o sentido de organizar e principalmente ter um local centralizado para a busca de informações necessárias no dia a dia, para nossos trabalhos.

Quando criamos um novo usuário, estamos utilizando o serviço de diretório, nesta base de dados (agenda), estamos guardando, nomes, sobrenomes, endereços, logins, senhas, grupos, ao qual o usuário pertence dentre outras tantas opções que podemos cadastrar, tudo isto ficará disponível dentro de uma base de dados, esta base de dados poderá ser utilizada pelos nossos servidores para vários trabalhos. Vamos citar três soluções de serviço de diretório:

* Open Ldap para Sistemas Open Source.
* EDirectory para Sistemas Novell.
* Active Directory para Sistemas Microsoft e com suporte para todos acima citados.

No mercado de hoje nossos negócios precisam ter informações rápidas, de fácil atualização, alta disponibilidade e principalmente muita segurança e o Active Directory pode nos oferecer todos estes atributos e muito mais...

O Active Directory assumiu o mercado de serviços de diretório pelo seu desempenho, segurança e principalmente disponibilidade, o Active Directory esta no mercado desde o lançamento do Windows 2000 Server, após o seu nascimento assumiu a liderança dos serviços de diretório, utilizando como base o LDAP e a comunicação através de replicação lançou vários atributos e principalmente ferramentas para facilitar o gerenciamento de informações nas empresas.

Hoje quando criamos um usuário para logar no domínio de nossa empresa, estamos utilizando um serviço de diretório e por consequência usando o Active Directory.
Abaixo temos uma figura para demonstrar todos os recursos que o Active Directory pode utilizar como serviço de diretório de sua empresa:

![SMB](https://raw.githubusercontent.com/lobocode/lobocode.github.io/master/post/images/samba/ad1.png#center)

A cima, comparamos o AD com uma agenda, o AD físicamente também tem um banco de dados, este banco é conhecido com **NTDS.dit** e esta localizado na pasta %SystemRoot%\NTDS\ntds.dit em uma instalação default do AD. Este diretório chamado de NTDS apenas existirá nos servidores que tenham a função de Domain Controllers (DC’s). Neste diretório existirão os arquivos relacionados abaixo. Durante o processo de instalação do Active Directory **da Microsoft**, são criados cinco arquivos:

* Ntds.dit - Arquivo de banco de dados do AD
* Edb.log - Arquivo onde são armazenados todas as informações de log das transações feitas no AD.
* Edb.chk - Arquivo de checkpoint controla transações no arquivo Edb.log já foram comitadas no arquivo Ntds.dit.
* Res1.log - Arquivo de reserva assegura que alterações sejam gravadas na base(Ntds.dit) no caso de falta de espaço em disco.
* Res2.log - Arquivo de reserva assegura que alterações sejam gravadas na base(Ntds.dit) no caso de falta de espaço em disco.

Bem, agora sabemos que a estrutura lógica do AD é gravada em uma base de dados física chamada de Ntds.dit, porém Domain Controller é Active Directory? **Claro que não**, no desenho abaixo demonstramos claramente a diferença entre AD (estrutura lógica do AD) e DC (Servidor que contém uma cópia do NTDS.dit do AD). No desenho abaixo imaginemos uma construção, estamos construindo um grande salão de festas. Imaginem que nosso teto (retângulo azul) é nosso Active Directory, porém precisamos apoiar este teto em pilares (cilindros vermelhos), caso contrário nosso teto irá desabar, certo?

![SMB](https://raw.githubusercontent.com/lobocode/lobocode.github.io/master/post/images/samba/ad2.png#center)

É isto mesmo, o Active Directory é a estrutura lógica (teto), e os DC’s são servidores fisicos (pilares), por isto a necessidade de termos muitos DC’s espalhados. Assim nosso AD mesmo na falha de um DC (pilar) ou vários DC’s ainda conseguirá responder as solicitações e pedidos de nossa infraestrutura.

![SMB](https://raw.githubusercontent.com/lobocode/lobocode.github.io/master/post/images/samba/ad3.png#center)

Isto só é possível porque cada servidor quando recebe a função de Domain Controller, herda a criação do diretório %SystemRoot%\NTDS\ e toda a estrutura comentada acima. Todos os dados criados originalmente são replicados para o novo DC criado. Assim em um AD (domínio) com três DC’s como na figura abaixo, todos os Dc’s estão atualizados com todos os dados igualmente, isto recebe o nome de replicação do Active Directory.

![SMB](https://raw.githubusercontent.com/lobocode/lobocode.github.io/master/post/images/samba/ad4.png#center)

O Serviço de Diretório do AD é divido em duas estruturas a estrutura lógica e a estrutura física, o conhecimento pleno sobre a estrutura do AD é muito importante, principalmente quando maior for sua estrutura. Nestas explicações informaremos algumas ferramentas que podem auxiliar na verificação deste processo, vale lembrar que estamos focando as explicações no Active Directory do Windows Server 2008 R2, porém estas explicações poderão ser utilizadas em qualquer uma das versões de Active Directory, nos próximos artigos falaremos das evoluções para as futuras versões.

Quando falamos de estrutura lógica do Active Directory, muitos termos são falados, a estrutura lógica do AD consiste em Objetos, Unidades Organizacionais, Domínio, Árvores de Domínio e Floresta. Utilizamos a estrutura lógica do AD para podermos gerenciar os objetos dentro da organização. Já os objetos, são os componentes mais básicos da estrutura lógica e representam, usuários, computadores e impressoras. 

No caso do OU (Unidades Organizacionais), é um objeto de container, utilizado para organizar outros objetos. A organização pode ser feita de várias formas.
Geográfica – Onde as OU’s representam Estadas ou Cidades de sua estrutura física Exemplo: OU SP - OU RJ Setorial – Onde as OU’s representam setores da estrutura física da empresa, por unidade de negócio. Exemplo: OU Administrativo – OU Produção Departamental – Onde as OU’s representam setores da estrutura física da empresa por departamento. Exemplo: OU RH – OU DP – OU Caldeira Híbrido – Modelo onde podemos interagir todos os modelos acima, na Figura abaixo temos um modelo disto.

![SMB](https://raw.githubusercontent.com/lobocode/lobocode.github.io/master/post/images/samba/ad5.png#center)

O domínio é a estrutura mais importante do Active Directory e tem 2 funções principais.

* Fecham um limite administrativo para objetos. “Quem esta fora não entra, quem esta dentro não sai”, claro que esta regra pode sofrer alteração media/sambante permissões de entrada e saída, como relações de confiança.
* Gerenciam a segurança de contas e recursos dentro do Active Directory

Vale lembrar que um domínio do Active Directory compartilham:

* Mesmo banco de dados Ntds.dit com cada Domain Controller dentro deste domínio.
* Diretivas de segurança.
* Relações de Confiança com outros domínios.

Quando precisamos criar um segundo domínio, na maioria das vezes por necessidades no processo de segurança temos o que chamamos de domínios filhos. Quando temos um domínio pai com seus domínios filhos, chamamos de árvore de domínio, pois dividem o mesmo sufixo DNS, porém em distribuição hierárquica. Abaixo colocamos um exemplo para ilustrar nossa explicação.

Criamos o domínio livemotion.local (Figura esquerda abaixo), para podermos configurar diretivas de segurança, em um dado momento, precisamos criar um domínio novo, que tenha acesso aos recursos do domínio livemotion.local, porém tenha suas próprias necessidades de segurança (Figura direita abaixo).

![SMB](https://raw.githubusercontent.com/lobocode/lobocode.github.io/master/post/images/samba/ad6.png#center)

Conforme as figuras acima, podemos ver que quando temos um domínio filho, imedia/sambatamente estamos vinculados a um domínio pai, e esta divisão hierárquica de nome chamamos de Árvore de Domínios.O primeiro domínio de uma Floresta, chamamos de Root Domain, a Floresta receberá o nome deste domínio, a floresta pode ser feita de um único domínio com também estar dividida com várias árvores dentro da mesma floresta.

![SMB](https://raw.githubusercontent.com/lobocode/lobocode.github.io/master/post/images/samba/ad7.png#center)

Quando falamos de estrutura física do Active Directory, alguns termos são utilizados, a estrutura física do AD consiste em Domain Controllers e Sites.
A estrutura física do AD é totalmente independente da estrutura lógica do AD. A estrutura física é responsável por otimizar o tráfego de rede e manter segurança em locais físicos distintos. Um Domain Controller ou DC tem a função de executar o Active Directory e também armazenar a base do Active Directory bem como Replicar esta base “alterações” com outros DC’s.

Quando falamos de Árvores de Domínio ou até mesmo Floresta, vale lembrar que um DC pode apenas suportar um único domínio. Para criar uma disponibilidade do Active Directory podemos ter mais de um DC, sendo assim num exemplo de 2 Dc’s temos a base do Active Directory sendo replicada de forma perfeita entre os dois Dc’s.
A base do Active Directory o NTDS.dit é divido em partições, conforme a figura demonstrada abaixo:

![SMB](https://raw.githubusercontent.com/lobocode/lobocode.github.io/master/post/images/samba/ad8.png#center)

Estas partições formam o arquivo NTDS.dit, este é replicado entre cada um dos DC’s de seu domínio, consequentemente o arquivo é replicado para cada DC, tendo todos os Dc’s sincronizados logo teremos um Active Directory saudável e que pode suprir a falha de um DC, sem afetar o serviço de diretório do domínio.

---

### Protocolos do Active Directory

* **LDAP**
&minus; O protocolo LDAP é um protocolo de comunicação desenvolvido para uso em redes TCP/IP. O protocolo LDAP define a forma como um cliente de diretório pode acessar um servidor de diretório e a forma como o cliente pode executar as operações de diretório e compartilhar dados de diretório. Os padrões LDAP são estabelecidos por grupos de trabalho da equipe Internet Engineering Task Force (IETF). O Active Directory implementa as especificações de rascunho de atributos LDAP e os padrões IETF para o LDAP versões 2 e 3. 

Como está implícito em seu nome, o LDAP foi desenvolvido como um método eficaz para o acesso a serviços de diretório sem a complexidade de outros protocolos de serviços de diretório. Como o LDAP define as operações que podem ser executadas para consultar e modificar informações em um diretório, e a forma como as informações em um diretório podem ser acessadas com segurança, você pode usá-lo para localizar ou enumerar objetos de diretório e para consultar ou administrar o Active Directory.

O projeto OpenLDAP foi iniciado em 1998 por **[Kurt Zeilenga](http://www.openldap.org/project/kurt/)**. O projeto começou como um fork do projeto LDAP a partir da Universidade de Michigan.

O **clapd** é um simples daemon LDAP Connectionless, escrito como parte do esforço de pesquisa da IBM e concebido para responder às solicitações básicas que um cliente Windows faz. O clapd foi reescrito no Samba4.

* **DNS**

&minus; O Domain Name System ( DNS ) é um sistema de gerenciamento de nomes hierárquico e distribuído para computadores, serviços ou qualquer recurso conectado à Internet ou em uma rede privada. Ele baseia-se em nomes hierárquicos e permite a inscrição de vários dados digitados além do nome do host e seu IP. Em virtude do banco de dados de DNS ser distribuído, seu tamanho é ilimitado e o desempenho não degrada tanto quando se adiciona mais servidores nele. Este tipo de servidor usa como porta padrão a 53. A implementação do DNS-Berkeley, foi desenvolvido originalmente para o sistema operacional BSD UNIX 4.3.

Um domínio do Active Directory é fortemente baseado em um domínio DNS, particularmente devido à forte integração entre o Kerberos e DNS, e o fato de que este permite uma mudança para a hierarquia de nomes. 

O servidor DNS traduz nomes para os endereços IP e endereços IP para nomes respectivos, e permitindo a localização de hosts em um domínio determinado. Num sistema livre o serviço é implementado pelo software BIND. Esse serviço geralmente se encontra localizado no servidor DNS primário. O servidor DNS secundário é uma espécie de cópia de segurança do servidor DNS primário. Assim, ele se torna parte necessária para quem que usar a internet de uma forma mais fácil e evita que hackers roubem seus dados pessoais.

* **SNTP**
&minus;  O NTP é um protocolo para sincronização dos relógios dos computadores baseado no UDP para sincronização do relógio de um conjunto de computadores em redes de dados com latência variável. O NTP permite manter o relógio de um computador com a hora sempre certa e com grande exatidão. Originalmente idealizado por David L. Mills da Universidade do Delaware e ainda hoje mantido por si e por uma equipa de voluntários, o NTP foi utilizado pela primeira vez antes de 1985, sendo ainda hoje muito popular e um dos mais antigos protocolos da internet.

* **Kerberos**
&minus;  Kerberos é o nome de um Protocolo de rede, que permite comunicações individuais seguras e identificadas, em uma rede insegura. O protocolo Kerberos previne Eavesdropping e Replay attack, e ainda garante a integridade dos dados. Seus projetistas inicialmente o modelaram na arquitetura cliente-servidor, e é possível a autenticação mutua entre o cliente e o servidor, permitindo assim que ambos se autentiquem.

> O Kerberos necessita que os relógios internos dos clientes estejam sincronizados com o dele. Na configuração padrão, é necessário que os relógios dos clientes não tenham uma diferença maior do que 10 minutos. Na prática, servidores NTP são utilizados para manter os relógios do servidor e dos clientes sincronizados.

> Heimdal Kerberos é uma implementação open source do protocolo Kerberos. 

* **SMB/CIFS**
&minus; O SMB/CIFS (Server Message Block/Common Internet File System) é um protocolo de redes cujo o uso mais comum como foi dito anteriormente é o compartilhamento de arquivos em uma LAN. Para mais detalhes sobre este protocolo, vá em **[Protocolos SMB e CIFS](#protocolos-smb-e-cifs)**. 

* **NetBIOS**
&minus;  É uma API que fornece serviços relacionados com a camada de sessão do modelo OSI, permitindo que os aplicativos em computadores separados se comuniquem em uma rede local, não podendo ser confundido, portanto, como um protocolo de rede. 

Sistemas operacionais mais antigos executavam o NetBIOS sobre o IEEE 802.2 e o IPX/SPX usando os protocolos NetBIOS Frames (NBF) e NetBIOS sobre IPX/SPX (NBX), respectivamente. Em redes modernas, o NetBIOS normalmente é executado sobre TCP/IP através do protocolo NetBIOS sobre TCP/IP (NBT). Isso resulta que cada computador na rede possua um endereço IP e um nome NetBIOS correspondente a um (possivelmente diferente) nome de hospedeiro.

* **DCE/RPC**
&minus; DCE-RPC é um padrão estabelecido há muito tempo para o funcionamento de chamadas de procedimento remoto (RPC), e é publicado de forma gratuita pelo Open Group. O DCE-RPC descreve não só o formato para as chamadas de função, mas também um número de transportes de rede.

Lista de abreviaturas e siglas:

* **LDAP**: Lightweight Directory Access Protocol (ou Protocolo Leve de Acesso a
Diretórios).
* **AD**: Active Directory (ou Diretório Ativo).
* **DC**: Domain Controler.
* **RFC**: Request for Comments (ou Pedido para Comentários).
* **DFS**: Distributed File System (ou Sistema de Arquivo Distribuído).
* **DNS**: Domain Name System (ou Sistema de Nomes de Domínio).
* **NetBIOS**: Network Basic Input/Output System (ou Sistema Básico de Entrada/Saída
de Rede).
* **UCL**: User Account Control (ou Controle de Conta de Usuário).
* **PDC**: Primary Domain Controller (ou Controlador Primário de Domínio).
* **GPO**: Group Policy Objects.
* **CN**: Common Name.
* **NFS**: Network File System.
* **NTP**: Network Time Protocol.
* **PDC**: Primary Domain Controller.
* **SMB**: Server Message Block.
* **NMBD**: Cuida da resolução de nomes nos PC's em seus endereços de IP. 
* **smbclient**: Cliente SMB para GNU/Linux, permite Pcs GNU/Linux acessarem Pc's GNU/Linux e servidores SAMBA.
* **testpam**: Testa o arquivo de configuração SAMBA caso tenha algum erro no smb.conf.
* **smbclient**: Listará as conexões ativas com o servidor e dará o status do serviço.
* **smbpasswd**: Permite alterar e adicionar senhas compatíveis com o padrão SMB.
* **Snap-in**: Ferramentas administrativas, aplicações modulares de outros aplicativos.

---

### Requisitos e Preparacao do Sistema Operacional

Por conveniência, irei trabalhar com Debian por tratar-se da distribuição GNU/Linux mais usada em servidores. Quanto à versão do Debian, é recomendado neste caso sempre a versão **Stable** visto que o ciclo de atualização do sistema Stable é diferente das versões unstable. Por exemplo, em sistemas stable, só haverá atualização de pacotes e sistema, caso já tenham esgotado todos os possíveis problemas, bugs relacionados.

> **Nota:** Para detalhes de compatibilidade com sua respectiva arquitetura no Debian, basta acessar a documentação **[https://www.debian.org/releases/stable/installmanual](https://www.debian.org/releases/stable/installmanual)**.

---

### Samba4 como AD

Uma dúvida bastante frequente entre os administradores de sistemas, é quanto à instalação do Samba com relação a forma de instalação se por código-fonte (source-code), ou do repositório através de pacotes pré-compilados no formato ".deb". Geralmente recomendo que procure os formatos pré-compilados pois, as vezes pode tornar-se uma dor de cabeça "atualizar ou remover" programas compilados diretamente no código-fonte. Mas se você se garante tanto com um, quanto com outro, então é irrelevante.

O que iremos instalar a partir de agora:

* **attr** - Utilitários para manipulação de atributos estendidos do sistemas de arquivos.
* **acl** - Utilitários da lista de controle de acesso.
* **krb5-user** - Programas básicos para autenticar usando o Kerberos.
* **ntp** - Daemon e programas utilitários do "Network Time Protocol".
* **samba** - Servidor de login, impressão e arquivos SMB/CIFS para Unix.
* **smbclient** - Clientes SMB/CIFS em linha de comando para Unix.

---

### Codigo fonte do Samba 

Tratando-se de código fonte, tanto faz se para Debian, CentOs ou outra distribuição. O importante é você saber baixar a versão mais recente e ao mesmo tempo compilar caso deseje prosseguir com este tipo de instalação:

**[Instalação do Samba pelo código-fonte](https://www.samba.org/samba/download/)**

Caso você deseje simplificar as coisas, sugiro que pule esta parte bem como da instalação através do código fonte propriamente.

---

### Instalando o Samba

```bash
sudo apt install attr acl krb5-user ntp
```

Deixe as respostas padrão na configuração do Kerberos. Apenas atentando para a primeira pergunta "Reino por omissão do Kerberos", informe seu domínio em letras maiúsculas.

```bash
sudo apt-get install -y samba smbclient
```

A instalação do **smbclient** é opcional, utilizaremos apenas para testar as configurações adiante. Por padrão o Debian não vem com suporte a ACL então refina isto na partição **/** através do comando a baixo:

```bash
sudo vim /etc/fstab 
```

É interessante reconfigurar o seu sistema de arquivos caso seja ext3 ou ext4 para rodar o samba com compatibilidade com as acls, sistema de segurança contra falhas de sobrecarga entre outros. Para isso, altere sua configuração de modo parecido com o abaixo (não copie e cole esta linha pois, cada máquina tem uma configuração de UUID diferente). Localize a variável **errors** na área **pass**, e acrescente o user_xattr, acl,barrier como a baixo:

```bash
UUID=3e0fbcf1-4eb5-4654-a5de-4775de85fc09 / ext4    user_xattr,acl,barrier=1,errors=remount-ro 0       1
```

O disco utilizado não apresenta partições separadas, apenas o /. Ajuste se necessário. Os parâmetros user_xattr, acl e barrier devem ser definidos na partição **onde os arquivos dos usuários serão alocados**.

> **Nota:** As opções acl e user_xattr são necessários, a fim de utilizar os recursos de arquivos POSIX. Preste atenção caso tenha definido a partição em Logical Volume Management (LVM). Pois, LVM não suporta a variável "barrier". O barrier serve para adicionar um sistema de segurança contra falhas de energia em formatações ext3 ou ext4.

Aplicar mudanças no fstab:

```bash
sudo mount -o remount,rw /
```

> **Nota:** O parâmetro -o, vem de --option e o rw vem de read/write. Consulte a manpage de cada comando para compreender do que se trata.

Agora vamos configurar o arquivo de hosts. Para isto, adicione esta linha no arquivo /etc/hosts:

```bash
sudo echo "192.168.1.100 debian.seudominio.com.br debian" >> /etc/hosts
```

Agora configure a interface de rede que no debian 8, é através do arquivo `/etc/network/interfaces`:

```bash
# Conexão automatica das interfaces no startup do S.O
auto eth0
iface eth0 inet static

# coloque o ip correspondente a sua rede local
address 192.168.1.100 
netmask 255.255.255.0

# verifique seu gateway e coloque aqui
gateway 192.168.1.1
dns-nameserver 192.168.1.1
dns-search seudominio.com.br
```

> **Nota**: Caso tenha dificuldade para compreender como funciona a interface de rede, dê uma lida neste **[material configuração da interface de rede no linux](http://www.profissionaisti.com.br/2011/07/configuracao-da-interface-de-rede-no-linux/)**.

Reinicie a interface:

```bash
sudo systemctl restart networking
```

Configure o DNS client através da sua interface de rede `sudo nmtui`, ou pelo `/etc/resolv.conf` que é gerado pelo Network Manager, e acrescente as seguintes linhas:

```bash
domain seudominio.com.br
search seudominio.com.br
nameserver 192.168.1.100
nameserver 192.168.1.1
```

O nameserver 192.168.1.100 será utilizado pelo samba, enquanto o 192.168.1.1 é o serviço de DNS da rede local, pode ser modificado para outro de sua escolha (ex: 8.8.8.8 - google). Agora temos que configurar serviço de NTP (atualização de data/hora), que se encontra em `/etc/ntp.conf`:

Gere um backup do arquivo `/etc/ntp.conf`, e coloque as seguintes linhas no arquivo:

```bash
server 127.127.1.0
fudge 127.127.1.0 stratum 10
server a.ntp.br iburst prefer
server 0.pool.ntp.org iburst prefer
server 1.pool.ntp.org iburst prefer
driftfile /var/lib/ntp/ntp.drift
logfile /var/log/ntp
ntpsigndsocket /var/lib/samba/ntp_signd/
restrict default kod nomodify notrap nopeer mssntp
restrict 127.0.0.1
restrict a.ntp.br mask 255.255.255.255 nomodify notrap nopeer noquery
restrict 0.pool.ntp.org mask 255.255.255.255 nomodify notrap nopeer noquery
restrict 1.pool.ntp.org mask 255.255.255.255 nomodify notrap nopeer noquery
```

Para instalar o samba, bastava apenas ter digitado `sudo apt install -y samba` como mostra no início. No entanto, preferí deixar algumas coisas prontas antes de prosseguir-mos. 

> **Nota:** **[O Network Time Protocol (NTP)](https://pt.wikipedia.org/wiki/Network_Time_Protocol)** é um protocolo de rede para sincronização do relógio entre sistemas e computadores além de  medir a latência da rede basicamente.

Dê um reboot no sistema e quando retornar, por favor teste se há conexão com a internet. Se houver conexão, está tudo em ordem. Se não, algo deu errado (provavelmente na configuração do ip no /etc/network/interfaces.) Mas não se preocupe. Se isso acontecer, sugiro que você verifique o problema no `syslog` do S.O:

```bash
sudo tail -n 500 /var/log/syslog |grep networking
```

Seja qual for o erro,só avance se tudo estiver funcionando até aqui.

---

### Samba como um controlador de dominio

Você deve ter ouvido falar que sempre antes de começar a configurar um sistema apartir de um arquivo, é interessante que você faça a cópia do mesmo para fins de segurança. Pois bem, faremos o mesmo para com o arquivo `smb.conf`:

```bash
sudo mv /etc/samba/smb.conf /etc/samba/smb.conf.bkp
```

> **Nota**: Neste caso, poderíamos ter usado o **cp** ao invés do **mv** para fazer o backup do smb.conf. No entanto, a intenção aqui é justamente remover o smb.conf criado a partir do apt.

Agora iremos configurar o samba como controlador de domínio:

```bash
samba-tool domain provision --use-rfc2307 --realm=SEUDOMINIO.COM.BR --domain=SEUDOMINIO
--server-role=dc --dns-backend=SAMBA_INTERNAL --adminpass='S@mb4s3rveR!'
```
Se der tudo certo, certamente que irá gerar uma saída longa. No entanto, a parte importante a se destacar é esta a baixo:

```bash
Server Role:	active directory domain controller
Hostname:		debian
NetBIOS Domain:	SEUDOMINIO
DNS Domain:		seudominio.com.br
```

> **Nota:** É de suma importância que a senha do --adminpass seja segura (com números,letras e caracteres especiais). Caso contrário, retornará um erro e não irá promover o samba. Dei preferência ao SAMBA_INTERNAL como sistema de configuração do DNS. Se você prefere usar o BIND9, dê uma estudada em **[Configure BIND as backend for SAMBA AD](https://wiki.samba.org/index.php/Configure_BIND_as_backend_for_Samba_AD)**. 

Uma breve explicação dos parâmetros usados:

* **domain provision** - Promover o primeiro controlador de domínio AD/DC (ou um existente). 
* **--use-rfc2307** - Ativa extensões NIS. Eles permitem uma gestão central do Unix atributos (UIDs, escudos, GIDs, etc.) dentro do Active Directory. Recomenda-se sempre habilitar esse recurso durante o provisionamento. Não há desvantagens em não usá-lo.
* **--realm** - Usado para você setar o kerberos real name do dominio (sempre em MAIÚSCULO).
* **--domain** -nome de domínio NT4 NetBIOS em maiúsculas utilizados pelo AD por razões de compatibilidade. Comprimento nome máxima: 15 caracteres.
* **--server-role** - Define como controlador de domínio
* **--dns-backend** - Aqui você escolhe o servidor se bind9 ou samba_internal.
* **--adminpass** - Senha do administrador do AD/DC.

Ajuste os parâmetros conforme suas necessidades, especialmente o realm, domain e adminpass. Agora vamos copiar a configuração do Kerberos gerada pelo samba:

```bash
sudo cp /var/lib/samba/private/krb5.conf /etc/
```

> **Nota:** Criamos uma cópia do krb5.conf na pasta /etc/ porque é a pasta padrão da configuração do Kerberos para com o Samba 4.

Inicie o samba: 

```bash
sudo systemctl start samba
```

Desative a política de atualização de senha para administrador (opcional):

```bash
sudo samba-tool user setexpiry administrator --noexpiry
```

Ajuste as configurações do samba, conforme suas necessidades no arquivo `/etc/samba/smb.conf`:


```bash
[global]
        workgroup = seudominio
        realm = seudominio.COM.BR
        netbios name = DEBIAN
        server role = active directory domain controller
        dns forwarder = 192.168.1.1
        idmap_ldb:use rfc2307 = yes

        # logs
        log file = /var/log/samba/log.%m
        max log size = 50


[netlogon]
        path = /var/lib/samba/sysvol/domarques.com.br/scripts
        read only = No

[sysvol]
        path = /var/lib/samba/sysvol
        read only = No

[home]
        path = /var/sambausers/
        read only = no
        browseable = no
```

Outra maneira de fazer o mesmo processo, é apenas digitando **testparm** no terminal. Esta é uma ferramenta do samba-tool e serve para fazer análise da sintaxe do smb.conf. Caso não exista smb.conf, ele gera um novo e baseando o workgroup nas configurações existentes do sistema. Crie o diretório dos arquivos dos usuários e aplique as novas configurações:

```bash
sudo mkdir -p /var/sambausers
```
Para aplicar novas configurações:
```bash
sudo smbcontrol all reload-config 
```

Caso não saiba como configurar seu `smb.conf`, consulte o site **[http://www.sloop.net/smb.conf.html](http://www.sloop.net/smb.conf.html)**. Depois de tudo a cima feito, vamos testar as configurações. Verifique versão do samba e smbclient:

```bash
sudo samba -V
sudo smbclient -V
```

Agora reboot a máquina e teste o Kerberos:

```bash
sudo reboot
sudo kinit administrator@seudominio.COM.BR
```

É muito comum após este comando aparecer o seguinte erro:

```bash
kinit cannot contact any kdc for realm
```

Caso ocorra tal erro, sugiro que você investigue no `/var/log/syslog` ou journaling:

```bash
tail -n 500 /var/log/syslog |grep kinit
```

Em caso positivo, irá retornar algo semelhante a isto:

```bash
warning: Your password will expire in 41 days on Sex 29 Jan 2016 12:18:08 BRST
```

Teste também as configurações de DNS do samba:

```bash
sudo host -t SRV _ldap._tcp.seudominio.com.br
sudo host -t SRV _kerberos._udp.seudominio.com.br
sudo host -t A debian.seudominio.com.br.
```

Caso ocorra tudo nos conformes, irá retornar algo semelhante a isto:

```bash
_ldap._tcp.teste.com.br has SRV record 0 100 389 debpdc.teste.com.br
_kerberos._udp.teste.com.br has SRV record 0 100 88 debpdc.teste.com.br
debpdc.teste.com.br has address 192.168.1.100
```
Teste a conectividade com o samba:
```bash
sudo smbclient -k //debian.seudominio.com.br/netlogon -c 'ls'
```
E por fim teremos algo semelhante a isto:
```bash
Domain=[TESTE] OS=[Unix] Server=[Samba 4.1.17-Debian]
.   D		   0  Fri Dec 18 11:16:51 2015
..	D		   0  Fri Dec 18 11:21:12 2015
	49788 blocks of size 524288. 44277 blocks available
```

---
### Compartilhamento com o servidor de impressao
Você poderá compartilhar as impressoras já configurados com o CUPS mas, tenha em mente que o Samba comunica com o CUPS via soquetes, portanto, você não precisa configurar qualquer permissão especial, além de uma diretiva Listen para o socket CUPS. Criaremos um diretório de **[spool](https://pt.wikipedia.org/wiki/Spooling)** de impressão, e definir as permissões corretamente. Este é destino onde o Samba irá armazenar arquivos temporários relacionados para imprimir documentos:

```bash
sudo mkdir /usr/local/samba/var/spool
sudo chmod 1777 /usr/local/samba/var/spool
```

> **Nota:** Você deve estar se perguntando o que significa este 1777 certo? A permissão 1777, este “1” a frente significa bitstickc. Em termos práticos, quer dizer exatamente isto **não deixe ninguém excluir esta pasta ou mudar a permissão”**.

Configurando o /etc/samba/smb.conf para ler o diretório de spooling adicionando o seguinte:

```bash
[printers]
    comment = All Printers
    path = /usr/local/samba/var/spool
    browseable = Yes
    read only = No
    printable = Yes
```

Por uma questão de conveniência, os clientes do Windows podem consultar o servidor que está compartilhando uma impressora para um driver de impressão. Para habilitar essa funcionalidade no Samba, temos de criar uma impressora especial de compartilhamento de arquivos. Vamos então criar um diretório de compartimento de impressão, e arquitetura de sub-diretórios:

```bash
sudo mkdir -p /usr/local/samba/var/print/{COLOR,IA64,W32ALPHA,W32MIPS,W32PPC,W32X86,WIN40,x64}
```
E novamente configurar no smb.conf:
```bash
[print$]
    comment = Point and Print Printer Drivers
    path = /usr/local/samba/var/print
    read only = No
```
Reinicie as configurações do samba:

```bash
sudo smbcontrol all reload-config
```
A partir de agora efetue login como um administrador de domínio em um computador cliente Windows e clique em Iniciar -> Executar "\\samba\". Na lista de ações, clique duas vezes "Impressoras e faxes" e em seguida, clique em Arquivo -> Propriedades do Servidor. Na guia Drivers, clique em "Adicionar …", em seguida, em "Next". Em seguida escolha o driver que você gostaria de instalar e clique em “Avançar”, escolha as arquiteturas dos drivers que você deseja instalar.

> **Nota:** Mais a frente estarei mostrando a melhor forma de interagir com o samba no Windows com printscreen, imagens etc... Por hora, vamos focar nas configurações.

--- 

# Lixeira do Samba
O Active Directory fornece recursos muito úteis para recuperação objetos excluídos. Dependendo de como você configurou o seu domínio, é possível restaurar um conjunto de atributos e dados com este recurso habilitado. Sempre que um objeto é excluído do Active Directory, ele é movido para um container escondido, chamado "Objetos Excluídos (CN = Deleted Objects, DC = samdom, DC = exemplo, DC = com). E estes objetos se mantém neste lugar por um determinado período (e é configurável). Após esse período, os objetos serão definitivamente excluídos (mas podem ser recuperados).

### O que **NÃO** pode ser recuperado?
Se você não tiver habilitado este recurso antes e posteriormente optar por habilitar, obviamente que tudo o que vem antes de ativar o recurso não pode ser recuperado (porque está fora da política do recurso). Ou ainda objetos renomeados também não poderão ser recuperados.
Adicione estas linhas no `/etc/samba/smb.conf`:

```bash
[share]
path = /data/share
vfs objects = recycle
recycle:repository = .recycle
recycle:keeptree = yes
recycle:versions = yes
```
Isto é, qualquer item deletado neste compartilhamento vai para o diretório .recycle. Podemos também implementar mais recursos para esta tarefa. Por exemplo, experimente adicionar as seguints linhas em seu [global] no `/etc/samba/smb.conf`:

```bash
vfs objects = recycle
recycle:facility = LOCAL1
recycle:priority = NOTICE
recycle:maxsize = 0
recycle:directory_mode = 0774
recycle:subdir_mode = 0774
recycle:keeptree = true
recycle:touch = true
recycle:versions = true
recycle:repository = /srv/samba/Recycle/
recycle:exclude = *.tmp, *.log, *.obj, ~*.*, *.bak, *.exe, *.bin
recycle:exclude_dir = tmp, temp, cache
create mask = 0774
directory mask = 0774
```
Agora vamos à explicação de cada variável a cima:

*  **vfs recycle** - intercepta pedidos de remoção e move os arquivos afetados para um repositório temporário ao invés de removê-los imedia/sambatamente. É basicamente o mesmo efeito que o delete no Windows. Ou seja, ele não remove completamente. Ao invés, é direcionado à lixeira.
* **recycle:facility = LOCAL1 e recycle:priority = NOTICE** - Voce está indicando que usará um arquivo de log para registrar a movimentação dessa lixeira, isso será feito por meio do syslogd do sistema.
* **recycle:maxsize = 0** - É definido em bytes o tamanho máximo de um arquivo que é destinado a lixeira, zero significa sem limites.
* **recycle:repository = /srv/samba/Recycle/** - Aqui é definido o local, pasta onde serão armazenados os dados da lixeira.
* **recycle:directory_mode = 0774, recycle:subdir_mode = 0774,recycle:keeptree = true,recycle:touch = tryyue** - Essas variáveis determinam as permissões dos diretórios, se é para guardar o nome da pasta de onde o arquivo fora excluído (keeptree), se a data do arquivo eliminado pode ser trocado pela data da exclusão (touch).
* **recycle:exclude = *.tmp, *.temp, *.log, *.ldb, *.o, *.obj, ~*.*, *.bak, *.iso e recycle:exclude_dir = tmp, temp, cache** - Determina respectivamente nome de arquivos e diretorios que deverão ser ignorados pela lixeira, isto é, se alguém excluir um .bak, ele terá sido excluido definitivamente.
* **recycle:versions = Yes** - Aqui você determina se haverá controle de versão, isto é, se um arquivo for sobregravado então a cópia antiga será enviada para a lixeira e se novamente sobregravar o mesmo arquivo, outra cópia será enviada para a lixeira.

---

# Auditando acessos 
O Samba oferece também um recurso de geração de log. Ele pode ser ativado adicionando as opções abaixo na seção [global] do smb.conf:
```bash
log level = 1
log file = /var/log/samba/log.%m
max log size = 50
```

> **Nota:** O log file e max log size já estava configurado nos passos a cima em **[Samba como um controlador de dominio](http://lobocode.github.io/2015/09/11/samba4.html#samba-como-um-controlador-de-dominio)**. Apenas adicionei o **log level**.

A opção **"log level"** indica o nível das mensagens (de 0 a 10), sendo que o nível 0 mostra apenas mensagens críticas, o nível 1 mostra alguns detalhes sobre os acessos e os demais mostram diversos níveis de informações de debug, úteis a desenvolvedores. A opção **"log file"** indica o arquivo onde ele será gerado e a **"max log size"** indica o tamanho máximo, em kbytes. A partir do Samba 3.04 foi incluído um módulo de auditoria, que permite logar os acessos e as modificações feitas de uma forma muito mais completa que o log tradicional. Isso é feito através do módulo **"full_audit"**, que (do ponto de vista técnico) funciona de forma similar ao módulo "recycle" usado pela lixeira.

O primeiro passo é ativar o módulo no `/etc/smb.conf` e em [global] como mostra a baixo:
```bash
vfs objects = full_audit
```
O próximo passo é definir quais operações devem ser logadas através da opção **"full_audit:success"**, como em:

```bash
full_audit:success = open, opendir, write, unlink, rename, mkdir, rmdir, chmod, chown
```
As opções que incluí no exemplo são open (ler um arquivo), opendir (ver os arquivos dentro de uma pasta), write (alterar um arquivo), unlink (deletar um arquivo), rename (renomear um arquivo), mkdir (criar um diretório), rmdir (remover um diretório), chmod (alterar as permissões de acesso de um arquivo) e chown (mudar o dono de um arquivo). Continuando a configuração, especificamos as informações que desejamos que sejam incluídas no log, usando a opção **"full_audit:prefix"**:
```bash
full_audit:prefix = %u|%I|%S
```
Por padrão, o módulo loga não apenas os acessos e modificações, mas também um grande volume de mensagens de alerta e erros gerados durante a operação. A opção **"full_audit:failure = none"** evita que estas mensagens sejam logadas, fazendo com que o log fique muito mais limpo e seja mais fácil encontrar as opções que realmente interessam:
```bash
full_audit:failure = none
```
Concluindo, especificamos o nível dos alertas, entre os suportados pelo syslog, como em **full_audit:facility = local1, e full_audit:priority = notice**. Juntando tudo, temos:
```bash
vfs objects = full_audit
full_audit:success = open, opendir, write, unlink, rename, mkdir, rmdir, chmod, chown
full_audit:prefix = %u|%I|%S
full_audit:failure = none
full_audit:facility = LOCAL1
full_audit:priority = notice
```
Em full_audit:prefix, adicionei %u, %I e %S que indicam o **usuário (%u)** que realizou a operação, o **endereço IP (%I)** do usuário e o **compartilhamento acesso (%S)**. Na diretiva full_audit:failure: none estamos indicando que não será registrada nenhuma operação (none) que tenha obtido como status de execução alguma falha.

A penúltima linha **full_audit:facility = LOCAL1** indica que iremos rotular as mensagens de log do Samba (especificamente do módulo VFS full_audit) para que sejam consideradas mensagens que são originadas do recurso de sistema chamado local1.

Esta configuração pode ser tanto incluída dentro da seção [global] (de forma que o log inclua os acessos e as alterações feitas em todos os compartilhamentos) quanto ser incluída apenas na configuração de um compartilhamento específico. Com isso, o Samba vai passar a gerar os eventos referentes aos acessos. Falta agora configurar o sysklogd (o serviço responsável pela geração dos logs do sistema), para logar os eventos, gerando o arquivo de log que poderá ser consultado. Para isso, abra o arquivo "/etc/syslog.conf" e adicione a linha abaixo:

```bash
local1.notice /var/log/samba-full_audit.log
```
> **Nota:** O "local1.notice" corresponde aos valores informados nas opções "full_audit:facility" e "full_audit:priority", enquanto o "/var/log/samba-full_audit.log" é o arquivo de log que será gerado.

Depois de concluída a configuração, reinicie os serviços e o log passará a ser gerado imedia/sambatamente:

```bash
sudo systemctl restart samba
sudo systemctl restart sysklogd
```
Dentro do arquivo, você verá entradas contendo a data e hora, o nome da máquina, o usuário, o IP da máquina, o nome do compartilhamento, a operação realizada e o nome do arquivo ou pasta onde ela foi realizada, como em:

```bash
Dec 23 15:21:15 m1 smbd_audit: lobo|192.168.1.100|arquivos|opendir|ok|.
Dec 23 15:21:29 m1 smbd_audit: lobo|192.168.1.100|arquivos|open|ok|r|addr.txt
Dec 23 15:21:34 m1 smbd_audit: lobo|192.168.1.100|arquivos|mkdir|ok|trabalho
Dec 23 15:21:36 m1 smbd_audit: lobo|192.168.1.100|arquivos|opendir|ok|trabalho
Dec 23 15:21:43 m1 smbd_audit: lobo|192.168.1.100|arquivos|open|ok|w|trabalho/Samba.sxw
Dec 23 15:21:44 m1 smbd_audit: lobo|192.168.1.100|arquivos|open|ok|w|trabalho/foto.jpg#center
```

O log conterá entradas referentes a todos os usuários e máquinas, mas é fácil ver apenas as entradas referentes a um determinado usuário, compartilhamento, endereço IP ou outro parâmetro qualquer ao listar o arquivo pelo terminal usando o grep, que permite mostrar apenas as linhas contendo determinados trechos. Mais opções e parâmetros usados para verificação de logs através do vfs full audit você poderá conferir na **[manpage do vfs_full_audit](https://www.samba.org/samba/docs/man/manpages-3/vfs_full_audit.8.html)**.

### Rotacionamento de logs

Para completar a configuração iremos implementar a política de rotacionamento de logs. Rotacionamento de logs é uma tarefa que é realizada pela ferramenta logrotate, e possui grande importância em servidores. A maioria dos arquivos de log em sistema Unix utilizam formato plain text e dependendo da demanda de usuários e recursos (arquivos/diretórios) acessados, bem como o período em que isto está ocorrendo, estes arquivos de texto irão crescer ao ponto de ocupar centenas a milhares de Megabytes, ao ponto de utilizar todo os espaço disponível para esta finalidade.

O rotacionamento de logs permite que, de acordo com a política empregada, um arquivo possa ser segmentado, bem como seus segmentos mais antigos serem compactados **(de modo a reduzir o espaço utilizado)**. Dentro de uma política de log, além de considerar o espaço em disco disponível, deve-se observar o período em que deseja-se armazenar estes arquivos. No caso do arquivo audit.log que estamos definindo como ponto para registrar as ações realizadas em cada arquivo/diretório, precisamos definir quantos dias, meses ou anos estes registros estarão disponíveis ou quantos segmentos irão existir do arquivo e que período (diariamente, semanalmente, etc) estes segmentos serão criados.

No caso vamos criar um arquivo de parâmetros que irá criar até 32 segmentos, e parte destes segmentos serão compactados. Já em relação ao período de criação destes segmentos ele será diário, bem como estes arquivos serão segmentos considerando que o segmento mais recente tenha atingido pelo menos 2 Megabytes, para que as entradas mais antigas destes arquivo sejam movidas para outro arquivo de segmento.

Crie um arquivo audit_samba.conf em /etc/logrotate.d com o seguinte conteúdo:

```bash
/var/log/samba/audit.log {
 dayly
 rotate 32
 minsize 2M
 notifempty
 missingok
 sharedscripts
 copytruncate
 compress
 delaycompress
 postrotate
 /bin/kill -HUP `cat /var/run/smbd.pid /var/run/nmbd.pid 2> /dev/null` 2> /dev/null || true
 endscript
}
```
Os elementos mais importantes deste arquivo são:

* **dayly** - realiza o rotacionamento/segmentação do arquivo diariamente);
* **rotate 32** - irá rotacionar e manter no máximo 32 segmentos (os segmentos mais antigos serão descartados);
* **minsize 2M** - somente irá rotacionar que o arquivo de segmento mais recente conter no mínimo 2M, do contrário não será segmento no dia presente;
* **compress** - como o próprio nome já diz, os segmentos serão compactados (por padrão utilizando o utilitário gzip, entretanto, este comportamento pode ser alterado com o parâmetro compresscmd);
* **delaycompress** - atrasa a compressão para o próximo rotacionamento, ou seja, teremos arquivos de segmento compactados a partir do antepenúltimo segmento mais recente até o segmento mais antigo.

Entretanto, ainda falta alterar um arquivo de políticas do logrotate chamado /etc/logrotate.d/samba.

Este arquivo contém uma política de rotacionamento que será aplicada a todos os arquivos com extensão log existentes em /var/log/samba, entretanto, como temos um arquivo de política já especifico para o arquivo audit_samba.log (que irá expandir muito mais rapidamente que qualquer outro arquivo de logs do samba), vamos adaptar o arquivo de políticas de rotacionamento padrão para não aplicar estas políticas no arquivo audit_samba.log.

Alterar o arquivo /etc/logrotate.d/samba, onde existe:
```bash
/var/log/samba/*.log
```
Altere para: 
```bash
/var/log/samba/[b-z]*.log
```

Diferente de outros utilitários do sistema, o logrotate **não é executado em background como um serviço, ele é executado media/sambante as políticas de agendamento de tarefas definidas no cron**, sendo assim, o utilitário logrotate será executado todos os dias a 4:02 da manhã de arquivo com o arquivo /etc/crontab e existência do arquivo de agendamento em /etc/cron.daily.

---

### Usando acls para permissoes avancadas no Linux

As ACLs (Access Control Lists) nos fornecem um controle mais refinado sobre quais usuários podem acessar diretórios e arquivos específicos do que as permissões tradicionais do GNU/Linux. Usando as ACLs, podemos especificar as formas nas quais cada um dos usuários podem acessar um diretório ou um arquivo. Se eu quiser dar uma permissão apenas para mais uma pessoa, por exemplo, a minha opção com chmod, seria criar um grupo de apenas uma pessoa e autorizar este grupo a acessar o arquivo ou diretório. Enfim, fica muito complicado quando precisamos regular o acesso de uma maneira mais detalhada. 

Existem dois tipos de ACLs: regras de acesso **(access ACLs)** e regras padrão **(default ACLs)**. Uma regra de acesso especifica informações de acesso para um único arquivo ou diretório. Já uma regra padrão é aplicada apenas a diretórios, e especifica informações de acesso padrões para todos os arquivos no diretório que não possuam uma ACL explícita aplicada. Como vimos no capítulo 4 em **[Instalando o Samba](http://lobocode.github.io/2015/09/11/samba4.html#instalando-o-samba)**, note que instalamos também as ACL's e ajustamos o particionamento para aceitar a acl no /etc/fstab. De agora em diante vamos trabalhar com uma maneira diferente de lidar com permissões além do já conhecido chmod.

Para usar este tipo de permissão, é bastante simples apesar de haver inúmeros parâmetros. Por exemplo, crie uma pasta chamada testes:

```bash
mkdir testes
```

Em seguida, vamos setar o acesso a leitura, escrita e gravação "apenas" para o usuário "lobo" (o meu usuário por exemplo):

```bash
sudo setfacl -m u:lobo:rwx testes
```

O comando que define as permissões chama-se **setfacl (set file access control list)**. Os parâmetros fornecidos também são de fácil compreensão:

* **-m** - Modifica as permissões de acesso do arquivo ou diretório.
* **u** - Modificações se aplicam a um usuário.
* **lobo** - Usuário que receberá as permissões de acesso.
* **rwx** - Estão sendo concedidas permissões de leitura (r), gravação (w) e execução (x).
* **testes** - Nome do diretório que somente o usuário "lobo" terá acesso.

Uma vez emitido o comando, vamos verificar se está tudo correto. Para isto usamos o comando `getfacl`:

```bash
$ getfacl testes
# file: testes
# owner: root
# group: root
user::rwx
user:lobo:rwx
group::r-x
mask::rwx
other::r-x
```

Como podemos ver, o dono do arquivo é o super usuário (root) e o usuário lobo recebeu permissões de leitura, escrita e execução neste arquivo (user:lobo:rwx). Para saber se um arquivo ou diretório possui uma lista de acesso, digite:

```bash
ls -l testes
```
E teremos algo semelhante a isto:

```bash
-rw-rwxr--+ 1 root root 0 Dec  4 19:06 testes
```

Observe o sinal de "+" em -rw-rwxr--+. Este sinal indica que o arquivo possui uma lista de acesso **(acl)** definida. Se emitirmos o mesmo comando para um diretório teremos um resultado diferente. Para explorar mais das acl's, sugiro que dêem uma lida neste artigo escrito por **[Luis Felipe Silveira](http://www.hardware.com.br/dicas/acl-linux.html)**.

Para que o samba consiga interpretar corretamente as ACL's, adicione o parâmetro **map acl inherit = Yes** no compartilhamento no qual você deseja ativar ACL (ou na pasta compartilhada que você alterou permissões e deseja efetivar as ACL) no smb.conf:

```bash
[compartilhado]
comment = compartilhado
path = /compartilhado
read only = No
create mask = 0777
force create mode = 0777
directory mask = 0777
force directory mode = 0777
map acl inherit = Yes
```
Não esqueça depois de alterar as ACL de reiniciar o samba para que ele aplique as configurações corretas:
```bash
sudo systemctl restart samba
```

---

### Migrando um samba3 PDC para Samba 4 AD

Recomendo fazer exaustivos teste em um ambiente virtual antes de fazer a migração no ambiente de produção. Para ter um ambiente de teste confiável apos criar as máquinas Virtuais ingressei as duas no meu domínio antigo em produção (Samba3+ldap) e me loguei em cada uma delas com pelo menos 4 usuários do domínio, diferente em cada maquina Totalizando 7 ( Um usuário em comum nas duas maquina para teste ). Você poderá criar uma situação similar para testes também. Acesse seu servidor samba3 e nele vamos fazer **Backup** de algumas pastas e arquivos de configurações que serão necessários para a migração:

```bash
sudo mkdir -p /root/backup/var/lib/
sudo mkdir -p /root/backup/etc/
```

Parando o servidor ldap do samba3 para fazer backup:

```bash
sudo systemctl stop ldap
```

Fazendo Backup do servidor ldap

```bash
suddo slapcat > /root/backup/backup.ldif
```

Iniciando o servidor Ldap (Para que seus usuários possam voltar a logar):

```bash
sudo systemctl start ldap
```

Copiar arquivo as pastas do samba:

```bash
sudo cp -r /etc/samba/ /root/backup/etc/
sudo cp -r /var/lib/samba /root/backup/var/lib/
```

Copiando pasta de configuração do ldap

```bash
sudo cp -r /etc/openldap /root/backup/etc/
```

No meu caso o ldap usa conexão segura TLS por isso precisei dos certificados também. Para facilitar copiei toda as pasta /etc/ssl:

```bash
sudo cp -r /etc/ssl /root/backup/etc/
```

Copiei a pasta Backup do servidor samba3 para o servidor samba4:

```bash
sudo scp -r /root/backup root@servidorsamba4:/root/
```

> **Nota:** Você poderá usar o filezila em ambiente gráfico se um dos servidores tiver o servidor X, ou qualquer outro recurso de ftp caso não saiba usar o **scp**.

Ligue as duas maquinas em rede (Samba3 e Samba4) recomendo que o teste de migração sejam feito numa rede separada simulando um cenário real (onde o samba4 tem o mesmo ip do antigo samba3). A instalação do Servidor openldap será apenas para a migração visto que já existe um servidor openldap embutido no samab4. Portanto, pós migração iremos remove-lo.

Antes de Instalar o ldap e fazer o dump dos usuários, verifique se existem usuários com sid duplicado e se existem grupos que tenha o mesmo nome que o de usuários.
Instalar servidor ldap:

```bash
sudo apt install -y libldap-2.4-2 slapd ldap-utils
```

> **Nota:** Quando for solicitado senha pode deixar em branco.

Parar o daemon do servidor ldap:

```bash
sudo systemctl stop slapd
```

Fazer copia da pasta padrão do servidor ldap (criada na instalação do servidor ldap):

```bash
sudo mv /etc/ldap /etc/ldap.padrao
```

Copiar a pasta do servidor ldap antigo ( Samba3 ) da qual fora feito o backup temporário:

```bash
sudo cp -r /root/backup/etc/ldap /etc/ldap
```

Mudando Permissões na pasta ldap

```bash
sudo chmod 777 -R /etc/ldap
```

Agora precisamos editar o arquivo `/etc/ldap/slapd.conf`:

```bash
# See slapd.conf(5) for details on configuration options.
# This file should NOT be world readable.
include /etc/ldap/schema/core.schema
include /etc/ldap/schema/cosine.schema
include /etc/ldap/schema/inetorgperson.schema
include /etc/ldap/schema/nis.schema
include /etc/ldap/schema/yast.schema
include /etc/ldap/schema/samba.schema
pidfile /var/run/slapd/slapd.pid
argsfile /var/run/slapd/slapd.args
modulepath /usr/lib/ldap/modules
access to attrs=SambaLMPassword,SambaNTPassword
by dn="cn=Administrator,dc=empresa,dc=casa" write
by * none
access to dn.base=""
by * read
access to dn.base="cn=Subschema"
by * read
access to attrs=userPassword,userPKCS12
by self write
by * auth
access to attrs=shadowLastChange
by self write
by * read
access to *
by * read

# BDB database definitions

allow bind_v2
loglevel 1024
TLSCertificateFile /etc/ssl/servercerts/servercert.pem
TLSCACertificatePath /etc/ssl/certs/
TLSCertificateKeyFile /etc/ssl/servercerts/serverkey.pem
database bdb
suffix "dc=empresa,dc=casa,"
rootdn "cn=Administrator,dc=empresa,dc=casa"
directory /var/lib/ldap
checkpoint 1024 5
cachesize 10000
index objectClass,uidNumber,gidNumber eq
index member,mail eq,pres
index cn,displayname,uid,sn,givenname sub,eq,pres
index sambaSID eq
index sambaPrimaryGroupSID eq
index sambaDomainName eq
moduleload back_bdb.la
```

O meu servidor ldap usa conexão segura TLS com isso temos que copiar os certificados:

```bash
sudo cp -r /root/backup/etc/ssl /etc/
```

Permissão na pasta ssl:

```bash
sudo chmod 777 -R /etc/ssl/
```

Adicionando a base ldap:
```bash
sudo slapadd -l /root/backup.ldif
```

Inciando o servidor ldap:
```bash
sudo systemcctl start slapd
```

Testando o Servidor ldap:

```bash
sudo ldapsearch -x -h 127.0.0.1
```

Pronto já temos um servidor ldap com nossa base de usuários. Lembre-se que o samba4 já tem um servidor ldap interno só estamos usando esse servidor ldap temporariamente para que possamos migrar as contas de usuários do samba3 (que usava ldap) para o samba4.
Fazendo a Migração:

```bash
sudo samba-tool domain classicupgrade --dbdir=root/backup/var/lib/samba/ --use-xattrs=yes 
--dns-backend=SAMBA_INTERNAL --realm=empresa /root/backup/etc/samba/smb.conf
```

Entendendo um pouco o faz cada parâmetro:

* **classicupgrade**:  A intenção desta função é a de fazer uma substituição completa de um domínio do Samba em estilo NT4 existente. É possível fazer a conversão e os usuários e máquinas vão simplesmente re-conectar à nova instalação AD Samba sem a necessidade manual deste procedimento.
* **--dbdir**: Caminho para o diretório que contém os arquivos de banco de dados tdb.
* **--use-xattr**: Usar o suporte ao sistema de arquivos subjacente para atributos estendidos.
* **--realm**: Você pode especificar o domínio na linha de comando, se ele já não estiver especificado no arquivo smb.conf do Samba3, no final adicionamos a localização do arquivo smb.conf.
* **--dns-backend**: Por padrão o Samba é configurado com o servidor DNS interno. Mas você poderá usar o BIND9 como explica no capítulo 4 em **[Samba como um controlador de dominio](ihttp://lobocode.github.io/2015/09/11/samba4.html#samba-como-um-controlador-de-dominio)**.

Apos migrar o domínio tive um problema na senha do meu usuário administrator com isso tive que setar a senha do mesmo novamente:

```bash
sudo samba-tool user setpassword Administrator
```

Removendo o servidor ldap do samba 4:

```bash
sudo aptitude purge slapd ldap-utils -y

```

Permissão na pasta dns do samba

```bash
sudo chmod 777 -R /opt/samba/private/dns
```

Por fim, no samba4 faça alguns testes do kerberos e do próprio samba4 que já foram descritos os passos no capítulo 4 em **[Samba como um controlador de dominio](http://lobocode.github.io/2015/09/11/samba4.html#samba-como-um-controlador-de-dominio)**, a partir de **Teste o Kerberos...**.

> **Nota:** Caso você tenha problemas com SID duplicados, por favor acessar a documentação do Samba em **[Prevent failure due to duplicate SID's](https://goo.gl/m1lyQJ)**. Isto é, lá contém um script em python que ajuda a resolver este problema.

Caso o script da documentação não funcione, você poderá verificar os SID duplicados antes de fazer o dump de usuários com o seguinte comando:

```bash
sudo cat /root/backup/backup.ldif | grep sambaSID | sort | uniq -d
```

Com isso, saberá quais os SID duplicados e poderá remover a todos manualmente em /etc/backup/backup.ldif tranquilamente. Caso você já tenha feito o dump de usuários, poderá usar o seguinte comando:

```bash
sudo slapcat | grep sambaSID | sort | uniq -d
```

Você poderá usar um interface gráfica para remover usuários do ldap como por exemplo, o **ldap-acount-manager - LAM** ou phpldapadmin.

---

### Samba-Tool

Samba-tool é a principal ferramenta de administração de usuários e contas do samba server. Com ela, poderemos adicionar, remover, editar, gerar parâmetros de expiração de senha, limitar usuário a uma determinada tarefa, criar grupos, lidar com grupos, criar um shellscript facilitador, usar por meio de um ambiente gráfico facilitador e muito mais. Como por exemplo:

Trocar senha do usuário:

```bash
sudo amba-tool user setpassword lobo --newpassword=1234.Mudar.Senha
```

Trocar senha do usuário e forca a troca no Próximo Login:

```bash
sudo samba-tool user setpassword lobo --newpassword=1234.Mudar.Senha --must-change-at-next-login
```

Deletar Usuário:

```bash
sudo samba-tool user delete lobo
```

Deletar Usuário e Deletar a sua pasta Home:

```bash
sudo samba-tool user delete lobo && rm -r /home/samba/lobo
```

Listar Todos os Usuários do samba:

```bash
sudo samba-tool user list
```

Desabilitar o Usuário com essa opção a conta não pode ser utilizada, mas permanece no servidor:

```bash
sudo samba-tool user disable lobo
```

Habilitar Usuário:

```bash
sudo samba-tool user enable lobo
```

A expiração de senha para todos os usuários do domínio e feita com outro comando essa altera somente do usuário especificado (bom para ser usado em certas exceções como por exemplo aquele diretor que insiste em ser uma exceção a regra) 10 e o numero de dias em que a senha ira expirar:

```bash
sudo samba-tool user setexpiry lobo --days=10
```

Desabilitar a expiração de senha:

```bash
sudo samba-tool user setexpiry lobo --noexpiry
```

Criar um grupo:

```bash
sudo samba-tool group add diretoria
```

Adicionar Vários Grupos de uma vez:

```bash
sudo samba-tool group add "diretoria diretoria_ead"
```

Criar um grupo e adicionar um descrição ao grupo:

```bash
sudo samba-tool group add diretoria --description="Grupo da diretoria"
```

Adicionar um membro a um grupo:

```bash
sudo samba-tool group addmembers diretoria lobo
```

No samba4 podemos adicionar um grupos dentro de outro isso é muito útil

```bash
sudo samba-tool group addmembers diretoria diretoria_ead
```

Adicionar Vários Membros a um grupo de uma vez só:

```bash
sudo samba-tool group addmembers diretoria "lobo,lobo2"
```

Remover um grupo:

```bash
sudo samba-tool group delete diretoria
```

Removendo Vários grupos de uma vez:

```bash
sudo samba-tool group delete "diretoria diretoria_ead"
```

Remover um membro de um grupo:

```bash
sudo samba-tool group removemembers diretoria lobo
```

Remover Membros de um grupo:

```bash
sudo samba-tool group removemembers diretoria "lobo,lobo2"
```

Listar todos os grupos:

```bash
sudo samba-tool group list
```

Listar Usuários pertencente a um grupo:

```bash
sudo samba-tool group listmembers diretoria
```

Mais sobre o Samba-Tool na **[documentação oficial](https://www.samba.org/samba/docs/man/manpages/samba-tool.8.html)**.

---

### Ingressando clientes no dominio Windows

Como o Windows 10 ainda é recente e o 8 não é tão prático quanto o 7, vamos nos focar no 7 para a configuração de domínio do samba. Lembrando que você poderá aplicar o mesmo no 8 e 10 que deverá funcionar. Acesse as propriedades do sistemas e em computador, clique no botão "Alterar". Informe o nome do domínio e clique em "OK". Será solicitado um nome e senha de um usuário do domínio, neste caso, iremos utilizar a conta de administrador do Samba, visto que ainda não criamos nenhum usuário comum. Será solicitado que o computador seja reiniciado como nos exemplos a baixo:
![Windows](https://raw.githubusercontent.com/lobocode/lobocode.github.io/master/post/images/samba/win7ad1.png#center)
![Windows](https://raw.githubusercontent.com/lobocode/lobocode.github.io/master/post/images/samba/win7ad2.png#center)
![Windows](https://raw.githubusercontent.com/lobocode/lobocode.github.io/master/post/images/samba/win7ad3.png#center)
![Windows](https://raw.githubusercontent.com/lobocode/lobocode.github.io/master/post/images/samba/win7ad4.png#center)

Ao efetuar este procedimento, o computador será automaticamente registrado no samba. Computadores com S.O. até o Windows XP serão registrados de forma diferente aos clientes com Windows Vista em diante, por conta das diferentes versões do protocolo SMB utilizadas. Reinicie o computador e troque o usuário para o administrador do domínio.

![Windows](https://raw.githubusercontent.com/lobocode/lobocode.github.io/master/post/images/samba/win7ad5.png#center)

> **Nota:** Caso não apareça "Fazer logon em: DOMÍNIO", informe o login como: DOMÍNIO\administrator.

---

### RSAT Ferramenta de administracao remota

Para administrar o samba pode-se utilizar o samba-tool, no entanto, recomenda-se a ferramenta **[Remote Server Admininstration Tools - RSAT da Microsoft](http://www.microsoft.com/downloads/details.aspx?FamilyID=7D2F6AD7-656B-4313-A005-4E344E43997D&displaylang=en)**. Após instalar, habilite as ferramentas necessárias para a administração do Samba, vá em painel de controle, "Programas e Recursos" e cliquem em "Ativar ou desativar recursos do Windows" (aba lateral esquerda)e selecione as ferramentas: 

* Ferramentas de Serviços da Área de Trabalho Remota;
* Ferramentas de Servidor DNS;
* Ferramentas do AD DS e AD LDS;
* Ferramentas de Servidor de NIS;
* Módulo do Active Directory para Windows PowerShell; e
* Ferramentas de Gerenciamento de Diretiva de Grupo.

![Windows](https://raw.githubusercontent.com/lobocode/lobocode.github.io/master/post/images/samba/win7ad6.png#center)

A principal ferramenta de administração do Samba da RSAT, é a "Usuários e Computadores do Active Diretory". Acessível em: "Painel de Controle" > "Ferramentas Administrativas". Ao executá-la, habilite a opção "Recursos avançados" para exibição plena dos recursos de administração.

![Windows](https://raw.githubusercontent.com/lobocode/lobocode.github.io/master/post/images/samba/win7ad7.png#center)
![Windows](https://raw.githubusercontent.com/lobocode/lobocode.github.io/master/post/images/samba/win7ad8.png#center)

Para outras versões do Microsoft Windows, consulte a **[instalação do RSAT na documentação do samba](https://wiki.samba.org/index.php/Installing_RSAT_on_Windows_for_AD_Management)**. Ao instalar e configurar o Samba, fora definido um compartilhamento denominado "home", que destina-se ao uso pessoal dos usuários. Não se trata da função de perfil remoto, mas apenas um diretório onde o usuário poderá utilizar para armazenar arquivos pessoais e acessar em qualquer computador do domínio.
É necessário ajustar algumas permissões diretamente no Samba. Para isso, efetue o login do usuário administrator no Windows e acesse "Gerenciamento do computador". Vá no menu "Ação" > "Conectar a outro computador..." e informe o nome do servidor samba, neste caso "debian".

![Windows](https://raw.githubusercontent.com/lobocode/lobocode.github.io/master/post/images/samba/win7ad9.png#center)
![Windows](https://raw.githubusercontent.com/lobocode/lobocode.github.io/master/post/images/samba/win7ad10.png#center)
![Windows](https://raw.githubusercontent.com/lobocode/lobocode.github.io/master/post/images/samba/win7ad11.png#center)

Ao conectar no servidor, vá em "Ferramentas do sistema" > "Pastas Compartilhadas" > "home". Configure nas abas "Permissões de compartilhamento" e "Segurança" os seguintes nomes e grupos de usuários:

**Permissões de compartilhamento:**

```bash
Usuários autenticados    Controle Total
SISTEMA                  Controle Total
Domain Admins            Controle Total
```

**Segurança:**

```bash
PROPRIETÁRIO CRIADOR    Controle Total
Usuários autenticados   Ler & executar, Listar conteúdo da pasta e Leitura
SISTEMA                 Controle Total
Administrator           Controle Total
Domain Admins           Controle Total
```

Você poderá customizar essas permissões de acordo com suas necessidades.

![Windows](https://raw.githubusercontent.com/lobocode/lobocode.github.io/master/post/images/samba/win7ad12.png#center)
![Windows](https://raw.githubusercontent.com/lobocode/lobocode.github.io/master/post/images/samba/win7ad13.png#center)
![Windows](https://raw.githubusercontent.com/lobocode/lobocode.github.io/master/post/images/samba/win7ad14.png#center)

Clique em "OK" para salvar.

Através da RSAT iremos criar um usuário comum, sem privilégios de administrador. Abra a ferramenta "Usuários e Computadores do Active Diretory". Em seguida vá no menu "Ação" > "Novo" > "Uusuário". Informe os dados do usuário, como nome, sobrenome, logon e senha. As senhas deverão atender a política de senhas do Samba, por padrão requer caracteres maiúsculos, minúsculos e números.

![Windows](https://raw.githubusercontent.com/lobocode/lobocode.github.io/master/post/images/samba/win7ad15.png#center)
![Windows](https://raw.githubusercontent.com/lobocode/lobocode.github.io/master/post/images/samba/win7ad16.png#center)
![Windows](https://raw.githubusercontent.com/lobocode/lobocode.github.io/master/post/images/samba/win7ad17.png#center)

Após criá-lo, consulte-o em "Usuários" e configure o compatilhamento "home", conforme:

![Windows](https://raw.githubusercontent.com/lobocode/lobocode.github.io/master/post/images/samba/win7ad18.png#center)
![Windows](https://raw.githubusercontent.com/lobocode/lobocode.github.io/master/post/images/samba/win7ad19.png#center)

Ao concluir, basta efetuar o login com o novo usuário criado:

![Windows](https://raw.githubusercontent.com/lobocode/lobocode.github.io/master/post/images/samba/win7ad20.png#center)

---

### Configurando perfil nomade
O Roaming Profile (Perfil Nomade) ou móvel, é criado pelo administrador do sistema e armazenado em um Servidor. Esse perfil está disponível sempre que você faz logon em qualquer computador na rede. Qualquer alteração feita no perfil Roaming Profile será atualizada no Servidor. Se o usuário efetuar logon em outra máquina, todas as configurações e preferências do Desktop (Área de trabalho), como por exemplo, impressoras, papel de parede, configurações de vídeo, etc, estarão disponível para o usuário.
Antes de iniciar-mos o processo, é interessante verificar algumas coisas como por exemplo, se o processo samba está rodando:

```bash
ps -ef |grep samba
```

Identificar em quais portas e interfaces o samba está escutando:

```bash
netstat -tunlp |grep samba
```

Agora crie um diretório chamado "userprofiles" dentro do "/" no servidor samba:

```bash
mkdir /userprofiles
```

Forneça permissão total ao diretório:

```bash
chmod 777 /userprofiles
```

Agora configure o diretório que acabamos de criar no smb.conf:

```bash
sudo vim /etc/samba/smb.conf
```

E edite com as seguintes linhas:

```bash
{% highlight bash %}
[Profiles]
path = /userprofiles
read only = no
valid users = lobo
{% endhighlight %}
```

Reinicie o samba:

```bash
sudo smbcontrol all reload-config
```

Teste o kerberos:

```bash
sudo kinit administrator@seudominio.com.br
```

Verifique se o diretório compartilhado está disponível:

```bash
sudo smbclient -L localhost -U%
```

Agora vá para o computador com Windows onde você instalou a ferramenta de administração remota (AD) para o seu servidor de domínio e defina os usuários a quem você deseja definir como perfil nomade. Abra o cmd e digite **dsa.msc**, ou run e digite **dsa.msc** para abrir console do Active Directory e em seguida defina o caminho do usuário:

![Windows](https://raw.githubusercontent.com/lobocode/lobocode.github.io/master/post/images/samba/win7ad23.png#center)

Vá para o usuário para o qual você deseja implementar perfil nomade e adicione o caminho seguido pelo nome do usuário do diretório do perfil no **profile path** da seção de propriedades como mostra abaixo:

![Windows](https://raw.githubusercontent.com/lobocode/lobocode.github.io/master/post/images/samba/win7ad24.png#center)

Daqui em diante você apenas acompanha se o perfil nomade está funcionando ou não. Uma forma de fazer isto, é acessando o usuário no Windows, fazer modificações de arquivos e pastas e acompanhar modificações na pasta /userprofiles no samba. 

> **Nota:** Certifique-se de que o usuário estará logando em máquinas onde o Sistema Operacional se mantém o mesmo. Isto é, que ele esteja logando em máquinas do Windows 7 ao invés de outras versões. Pois, pode ser que não funcione caso o usuário fique transitando entre uma versão e outra do mesmo sistema. Caso ainda tenha dificuldades em operar com perfil nomade no windows7, dê uma olhada **[neste vídeo](http://samba.org/tridge/Samba4Demo/s4demo3.ogv)**. Recomendo que tenha o VLC player instalado na máquina.

---

### Adicionando unidades organizacionais 

Para trabalharmos com o unidades organizacionais (UO) no samba, poderemos fazer isto usando o **[snap-in](https://msdn.microsoft.com/en-us/library/bb742437.aspx#EEAA)** do Windows, ou até mesmo o Zentyal, GOsa2, Webmin, Smb4k, o Lam, SMB2WWW, gnomba, Jabs, Komba2, Smb Web client e muitos outros. No entanto, como a maioria dos administradores de sistemas estão mais acostumados a trabalhar com o **[snap-in](https://msdn.microsoft.com/en-us/library/bb742437.aspx#EEAA)** do Windows, a este darei preferência por enquanto. Mais a frente farei uma abordagem ao GOsa2 e outras tecnologias.

Este costuma ser um procedimento bastante simples como mostra a seguir:

* Menu iniciar > iniciar > dsc.msc.
* Clique com o botão direito do mouse em seu domínio.
* Escolha a opção new ou (novo), e vá em organizational unit (unidade organizacional).
* Em type, coloque o nome 'OU Demo' por exemplo.
* Em seguida será exibida uma unidade organizacional chamada 'OU Demo'. 

![Windows](https://raw.githubusercontent.com/lobocode/lobocode.github.io/master/post/images/samba/win7ad25.png#center)

A partir de agora, você poderá mudar o lugar onde criou a unidade organizacional ou simplesmente criar sub unidades organizacionais.

> **Nota:** Normalmente nós criamos UO baseado na configuração dodepartamento de sua organização. Tenha cuidado para não confundir grupos e unidades organizacionais, grupos são usados para controlar permissões, **OU são utilizados para as configurações de implantação para todos os usuários/computadores dentro da UO**.

---

### Implementando Group Policies

Assim como para criar unidades organizacionais usando o snap-in do Windows, para criar GPO's também é bastante simples. Veja os exemplos a baixo:

* Clique em Iniciar, Ferramentas Administrativas e Diretivas de Grupos.
![Windows](https://raw.githubusercontent.com/lobocode/lobocode.github.io/master/post/images/samba/win7ad26.png#center)
* Clique sobre com botão direito e  Criar um GPO neste domínio e …
![Windows](https://raw.githubusercontent.com/lobocode/lobocode.github.io/master/post/images/samba/win7ad27.png#center)
* Nome da GPO será “Exemplo_GPO” e OK.
![Windows](https://raw.githubusercontent.com/lobocode/lobocode.github.io/master/post/images/samba/win7ad28.png#center)
* Sobre a nova GPO clique com botão direito,  clique em  editar:
![Windows](https://raw.githubusercontent.com/lobocode/lobocode.github.io/master/post/images/samba/win7ad29.png#center)
* Para bloquear o acesso ao Painel de Controle vai em, Configurações do Usuário, Diretivas, Modelos Administrativos, Painel de Controle e  Proibir acesso ao Painel de Controle:

![Windows](https://raw.githubusercontent.com/lobocode/lobocode.github.io/master/post/images/samba/win7ad30.png#center)

Duplo clique sobre a diretiva e marque a opção  “Habilitado” de OK. Pronto, GPO criada com sucesso. Caso ainda existam dúvidas a respeito da criação de GPO's, dê uma olhada neste **[video da documentação oficial](http://samba.org/tridge/Samba4Demo/s4demo2.ogv)**.

---

### Backup e recovery do Samba AD

Aqui vai uma dica bacana para quem instalou o samba a partir de pacotes pré compilados via apt-get, dentro do source code, existe uma pasta que contém um script de backup chamado samba_backup que se encontra exatamente em ../source4/scripting/bin/samba_backup e vamos usar ele para este propósito. Copie este script para /usr/sbin/, mudar o proprietário, e em seguida a licença:

```bash
sudo cp ..../source4/scripting/bin/samba_backup /usr/sbin
sudo chown root:root /usr/sbin/samba_backup
sudo chmod 750 /usr/sbin/samba_backup
```

Dentro do script backup_samba ajuste as variáveis de acordo com suas necessidades:

```bash
FROMWHERE=/usr/local/samba
WHERE=/usr/local/backups
DAYS=90
```

Configure a pasta destino para onde será realizado o backup **dentro da variável WHERE=**:

```bash
sudo mkdir /usr/local/backups
sudo chmod 750 /usr/local/backups
```

Faça um tste de execução do script:

```bash
suddo /usr/sbin/samba_backup
```

> **Nota:** Se ocorrer algum erro, verifique se na pasta destino se encontra três arquivos. São eles: etc.{Timestamp}.tar.bz2, samba4_private.{Timestamp}.tar.bz2 e sysvol.{Timestamp}.tar.bz2. Em caso de não haver, busque pelos três no source code e coloque-os lá.

E em caso de sucesso, agora só nos falta adicionar o script ao crontab:

```bash
sudo crontan -e 
```

E a rotina de backup. Por exemplo para ele rodar sempre as 2am 

```bash
0 2 * * * /usr/sbin/samba_backup
```

Para restaurar o backup, ou melhor, fazer o recovery, é tão simples quanto os passos a cima. Primeiro vamos remover algumas pastas do samba:

```bash
sudo rm -rf /usr/local/samba/etc
sudo rm -rf /usr/local/samba/private
sudo rm -rf /usr/local/samba/var/locks/sysvol
```

Lembra dos três arquivos citados na **nota** a cima? Vamos usalos agora:

```bash
cd /usr/local/backups
sudo tar -jxf etc.{Timestamp}.tar.bz2 -C /usr/local/samba/
sudo tar -jxf samba4_private.{Timestamp}.tar.bz2 -C /usr/local/samba/
sudo tar -jxf sysvol.{Timestamp}.tar.bz2 -C /usr/local/samba/
```

Renomeie os arquivos .ldb.bak gerados para .ldb .Para tal, usaremos o find:

```bash
sudo find /usr/local/samba/private/ -type f -name '.ldb.bak' -print0 | while read -d $'\0' f ; 
do mv "$f" "${f%.bak}" ; done
```

Agora faça um upgrade no samba:

```bash
sudo samba_upgradedns --dns-backend=SAMBA_INTERNAL
```

Recovery efetuado com sucesso.

---

# Samba como servidor secundario

Vamos supor que você já tenha um domínio primário configurado (que pode ser Samba4 ou Windows Server 2008 AD). Mas você precisa criar um domínio secundário e migrar toda a base de dados do primário para o secundário (para dividir tarefas, **replicar a base de dados em outro servidor de domínio**, ou simplesmente corrigir algum problema do servidor primário). Vou então criar uma situação hipotética para ficar mais fácil explicar como funciona. São eles os domínios:
**Controlador de domínio primário:**

```bash
192.168.1.6
hostname: test.dominio.com
```
**Controlador de domínio secundário:**
```bash
192.168.1.5
hostname: test1.dominio.com
```

Com seu controlador primário já configurado, vá ao secundário em **/etc/resolv.conf** e mude o DNS para o do servidor primário:

```bash
search dominio.com
nameserver 192.168.1.6
```

Para preparar seu servidor de domínio secundário, você terá que configurar o Samba 4 em seu servidor atual também. Para tal, basta seguir normalmente a instalação já abordada nesta documentação quanto ao Samba 4. No entanto, você irá mudar somente o modo com que **promove** o samba:

```bash
sudo samba-tool domain join dominio.com DC -Uadministrator --realm=dominio.com --use-ntvfs
```

Assim, conseguiremos unir o servidor primário com o secundário. Agora, o próximo passo consiste em certificar que seu hostname do domínio secundário é resolúvel:

```bash
nslookup test1.dominio.com 
```

> **Nota:** Neste caso, depende muito do servidor DNS que você está adontando. Isto é, o BIND9 ou o Samba_internal.

Caso o seu DNS não esteja resolúvel, então vamos fazer a entrada  para o mesmo no servidor DNS do samba. Edite o arquivo `/etc/local/samba/private/dns/dominio.com` e adicione a seguinte linha:

```bash
test1 IN A 192.168.1.5
```

Salve, saia e reinicie o serviço (caso seja BIND9):

```bash
sudo systemctl restart named
```

E tente novamente testar o DNS:

```bash
nslookup test1.dominio.com
```

Teste também se o objectGUID é resolúvel:

```bash
ldbsearch -H /usr/local/samba/private/sam.ldb '(invocationid= * )' --cross-ncs objectguid 
```

Se tudo correr bem, deverá aparecer algo semelhante a isto:

```bash
# record 1
dn: CN=NTDS Settings,CN=TEST,CN=Servers,CN=Default-First-Site-Name,CN=Sites,CN=Configuration,
DC=dominio,DC=com
objectGUID: 74b975bc-c25c-4ce7-9773-fe3f6eb1b903
# record 2
dn: CN=NTDS Settings,CN=TEST1,CN=Servers,CN=Default-First-Site-Name,CN=Sites,CN=Configuration,
DC=dominio,DC=com
objectGUID: 607bc2dc-0754-49e3-aa37-9be403d0cc33
```

> **Nota:** Observe o **objectGUID** para o test1, isto é, para o servidor secundário.

Agora, este objectGUID deve ser também resolúvel:

```bash
host -t CNAME 607bc2dc-0754-49e3-aa37- 9be403d0cc33._msdcs.example.com
```

Se não estiver, você poderá atualizar o registro DNS usando o comando a baixo:

```bash
samba-tool dns add IP-of-your-DNS _msdcs.samdom.dominio.com607bc2dc-0754-49e3-aa37-9be403d0cc
33 CNAME test1.dominio.com -Uadministrator
```


Se ocorrer algum erro, tente novamente configurar manualmente no dns do samba editando o arquivo `/usr/local/samba/private/dns/dominio.com.zone`:

```bash
607bc2dc-0754-49e3-aa37-9be403d0cc33._msdcs.example.com. IN CNAME test1
```

> **Nota:** Não copie e cole estes comandos visto que cada objectGUID se difere de máquina para máquina. Observe o seu e use o exemplo a cima apenas como base.

Reinicie o named:

```bash
sudo systemctl restart named
```

> **Nota:** Novamente, se estiver usando o Samba_Internal, estes passos para com o teste do DNS são descartáveis e provavelmente ocorra tudo na mais perfeita ordem.

Agora tente novamente o comando:

```bash
host -t CNAME 607bc2dc-0754-49e3-aa37-9be403d0cc33._msdcs.example.com
```

E provavelmente dê tudo certo.

Finalmente atualize o IP do seu servidor de domínio secundário editando o arquivo `/etc/resolv.conf` do mesmo servidor que para o nosso caso é 192.168.1.5:

```bash
search dominio.com
nameserver 192.168.1.6
nameserver 192.168.1.5
```

### Testes de replicacao de diretorios

Agora é hora de ver se a replicação está funcionando para ambos os controladores de domínio. Assim que se você fizer qualquer alteração em um dos dc o mesmo deve refletir sobre o outro dc. Para verificar isto basta usar o comando a baixo:

```bash
samba-tool drs showrepl
```

Na máquina com Windows Server, caso você deseje gerenciar o servidor secundário a partir do **snap-in** do AD do Windows, abra o **Console de gerenciamento do Active Directory**, vá em **Ação** e selecione **Alterar controlador de domínio** como mostra a baixo:

![Windows](https://raw.githubusercontent.com/lobocode/lobocode.github.io/master/post/images/samba/win7ad31.png#center)

Aqui você deve ser capaz de ver os seus controladores de domínio disponíveis e seus status como mostrado abaixo:

![Windows](https://raw.githubusercontent.com/lobocode/lobocode.github.io/master/post/images/samba/win7ad32.png#center)

Agora teste criar um usuário no servidor secundário e acompanhar no primário como já mostrei no capítulo 6 em **[Samba tool](http://lobocode.github.io/2015/09/11/samba4.html#samba-tool)**. Se o usuário criado no servidor secundário aparecer no primário, a nossa replicação está funcionando bem. O mesmo a partir de qualquer controlador de domínio.

**Referências:**
<dl>
<dt><a href="https://www.packtpub.com/web-development/implementing-samba-4">Implementing Samba 4</a></dt>
<dt><a href="http://www.amazon.com/Active-Directory-Cookbook-Cookbooks-OReilly/dp/1449361420">Active Directory Cookbook</a></dt>
<dt><a href="https://openlibrary.org/books/OL12369507M/The_Definitive_Guide_to_Samba_4">The Definitive Guide to Samba 4</a></dt>
<br/>
</dl>

