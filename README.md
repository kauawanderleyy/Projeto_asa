# Projeto_asa

DOCUMENTAÇÃO DO PROJETO DE INFRAESTRUTURA E DEVOPS

Esta documentação apresenta o planejamento, a arquitetura de rede, o escopo dos serviços e os procedimentos de execução do laboratório de infraestrutura ágil automatizado via Vagrant e Ansible.


Identificação do Projeto

Campo                     Descrição
Integrantes da Equipe     Kauá e Pedro
Disciplina                Arquitetura de Sistemas Abertos (ASA)  
Instituição               Instituto Federal da Paraíba (IFPB)
Professor                 LEONIDAS LIMA


Descrição Geral do Funcionamento

O projeto consiste no provisionamento automatizado de uma infraestrutura de rede e servidores composta por 4 Máquinas Virtuais (VMs) baseadas no sistema operacional Linux Debian, orquestradas pelo Vagrant (VirtualBox) e configuradas dinamicamente através do Ansible.

A arquitetura baseia-se em um modelo centralizado onde um servidor de infraestrutura (arq) gerencia os serviços vitais da rede local (DHCP, DNS e armazenamento NFS). Isso permite o deploy isolado e seguro de um servidor de aplicação web (app), um servidor de banco de dados (db) e uma estação de gerenciamento cliente (cli), que valida os serviços através de interface gráfica.


Topologia e Especificação das Máquinas

arq: 192.168.56.112 / Servidor de Arquivos e Infraestrutura. Executa o DHCP Server, DNS Bind9, NFS Server e gerencia o pool de armazenamento LVM de 15GB.

app: 192.168.56.130 / Servidor de Aplicação. Executa o Apache2 Web Server hospedando a página oficial do grupo. Atua como cliente Autofs.

db: 192.168.56.120 / Servidor de Banco de Dados. Executa o MariaDB Server e atua como cliente Autofs para montagem sob demanda

cli: Dinâmico da LAN / Estação Cliente. Configurada com suporte gráfico X11 e Mozilla Firefox para renderização e teste local da página web.


Detalhes de Execução dos Playbooks


A automação foi dividida em playbooks modulares para garantir que a infraestrutura seja construída em camadas lógicas:

01_setup_repo.yml: 
(Configuração de Repositórios): Padroniza e atualiza os espelhos do gerenciador de pacotes APT em todos os nós, garantindo que dependências futuras sejam baixadas sem quebras.





02_hardening_usuarios.yml:
(Segurança, Sudo e Rede): Cria o grupo devops e os usuários customizados para os integrantes (Kauá e Pedro). Aplica o hardening do servidor OpenSSH ao injetar chaves públicas autorizadas (authorized_key) e desativar a autenticação por senha (PasswordAuthentication no). Também reinicia as interfaces de rede para validação de IP.


03_raid_lvm.yml: 
(Provisionamento de Armazenamento): Formata discos adicionais atrelados ao nó arq, criando um Volume Group e um Logical Volume (LVM) dedicado de 15GB para centralização de dados compartilhados.


04_setup_dhcp.yml:
(Automatização de Rede IP): Instala e ativa o servidor ISC DHCP no nó arq. Mapeia estaticamente as tabelas de endereços MAC das interfaces internas das outras VMs, amarrando de forma estática os IPs da matrícula (.130 para o app e .120 para o db).


05_setup_dns.yml:
(Resolução de Nomes Interna): Configura o servidor de nomes Bind9 no arq. Estabelece a zona autoritativa do domínio do projeto, permitindo que a URL oficial http://app.kaua.pedro.devops resolva diretamente no IP do servidor Apache.



06_setup_nfs.yml:
(Compartilhamento e Montagem Inteligente): Exporta diretórios de armazenamento do arq usando NFS com as flags de segurança all_squash (rebaixando acessos para o ID sem privilégios nfs-ifpb). Nos clientes (app e db), configura o Autofs em Modo Mapeamento Direto (/-) para montar a pasta sob demanda somente ao ser acessada, otimizando os recursos do sistema.




07_servicos_aplicacoes.yml: 
(Deploy das Aplicações Finais): Realiza a instalação final dos serviços de software: servidor Apache2 no nó de aplicação com o código HTML contendo as informações da equipe, o MariaDB no banco de dados e os pacotes gráficos (X11/Firefox) no nó cliente.


Histórico de Engenharia e Resolução de Problemas Críticos

Durante a homologação da infraestrutura em ambiente de testes, dois incidentes críticos foram identificados e solucionados de forma cirúrgica:

Bloqueio de Máscara do Autofs no Diretório /var: Inicialmente, o Autofs foi configurado capturando o caminho relativo /var. Isso causou uma sobreposição fantasma do kernel sobre o sistema de arquivos local, ocultando subdiretórios essenciais como /var/lib/apt/lists/partial e bloqueando o funcionamento do gerenciador apt. A falha foi sanada aplicando um lazy unmount (umount -l) para liberar o kernel, recriando as estruturas temporárias e migrando a configuração definitiva do Autofs para o Mapeamento Direto (/- apontando fixamente para /var/nfs), o que blindou o sistema operacional contra mascaramentos.



Dependência Crítica de Criptografia (Falta de Módulo Passlib): No Playbook 02, o Ansible falhou ao tentar processar o filtro de hashing password_hash('sha512') devido à ausência do pacote Python passlib na máquina controladora. A arquitetura do projeto foi otimizada convertendo as strings de senha puras em hashes SHA-512 pré-calculados diretamente inseridos no código YAML do playbook, tornando a execução 100% independente de pacotes locais na controladora do laboratório.



Instruções de Execução e Validação

Para inicializar o laboratório do zero absoluto no ambiente acadêmico, os seguintes comandos devem ser executados na pasta raiz do projeto:

# 1. Subir a infraestrutura de hardware (VMs limpas)
vagrant up

# 2. Executar a cadeia sequencial de provisionamento do Ansible
ansible-playbook playbooks/01_setup_repo.yml
ansible-playbook playbooks/03_raid_lvm.yml
ansible-playbook playbooks/04_setup_dhcp.yml
ansible-playbook playbooks/02_hardening_usuarios.yml
ansible-playbook playbooks/05_setup_dns.yml
ansible-playbook playbooks/06_setup_nfs.yml
ansible-playbook playbooks/07_servicos_aplicacoes.yml

