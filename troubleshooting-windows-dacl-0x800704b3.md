# 🛠️ Case de Troubleshooting: Restauração de Permissões DACL no Windows 11 e Erro 0x800704b3



GUIA TÉCNICO DE REPARO E RECONSTITUIÇÃO DE PERMISSÕES NTFS
Data de Execução: Agosto de 2026

Ambiente: Windows 11 Pro / Home (x64)

Escopo: Recuperação de Acesso à Raiz (C:\) e Correção de Erros de Permissão em Cadeia

1. INTRODUÇÃO
Alterações inadvertidas na Lista de Controle de Acesso Discrecional (DACL) da raiz da unidade principal (C:\) causam o bloqueio total do sistema de arquivos no Windows. Esse tipo de falha impede que tanto usuários comuns quanto administradores acessem o disco local, gerando um efeito cascata em que o sistema operacional perde a capacidade de ler seus próprios executáveis internos. O objetivo deste documento é detalhar o diagnóstico e o procedimento técnico utilizado para redefinir as permissões NTFS do volume e restaurar o sistema sem a necessidade de formatação.

2. DESENVOLVIMENTO
O bloqueio na raiz do volume C:\ impede a execução de processos essenciais do sistema operacional localizados em diretórios como C:\Windows\SystemApps. Como consequência, a camada de abstração do Windows falha ao tentar acessar binários locais e gera falsos-positivos de erro de rede (código 0x800704b3 - "O caminho da rede digitado está incorreto"). Simultaneamente, a perda de herança de permissões corrompe a estrutura de índices do diretório oculto da Lixeira ($Recycle.Bin), exibindo alertas de integridade.

Como o ambiente gráfico bloqueia a elevação de privilégios na sessão normal (retornando o Erro de Sistema 5 - Acesso Negado), a solução exige o isolamento do sistema operacional através do ambiente de recuperação (WinRE). No Modo de Segurança com Prompt de Comando, o sistema operacional carrega com privilégios elevados nativos, permitindo reestruturar a tabela de proprietários via utilitário takeown e reinjetar as regras de herança de contêineres e objetos (OI)(CI) aos grupos Administradores e Todos por meio do comando icacls.

3. SUMÁRIO EXECUTIVO
Durante uma manutenção de segurança, a DACL da unidade principal (C:\) foi alterada, resultando na perda dos privilégios de leitura e gravação para os grupos de Administradores e Usuários. O bloqueio impediu a execução de processos essenciais (como executáveis em SystemApps), gerando falsos-positivos de rede (0x800704b3) e a corrupção do índice da Lixeira ($Recycle.Bin). Este documento descreve a metodologia aplicada via linha de comando para redefinir a propriedade e as permissões de segurança NTFS sem formatar o volume.

4. SINTOMAS E DIAGNÓSTICO
Acesso Negado à Unidade Principal: Bloqueio imediato ao tentar abrir o Disco Local (C:\) via Explorador de Arquivos.

Bloqueio de Elevação de Privilégios: Erro de Sistema 5 (Acesso Negado) ao tentar executar o Prompt de Comando ou PowerShell elevado na sessão normal.

Falso-Positivo de Rede (Erro 0x800704b3): Falha ao carregar binários locais, interpretada pelo sistema como erro de mapeamento de rede.

Inconsistência na Lixeira: Alerta de que a Lixeira da unidade C:\ estava danificada.

📌 COMO DESCOBRIR O NOME DA SUA MÁQUINA
O nome do computador (hostname) serve para identificar seu equipamento na rede e no próprio Windows. Para descobrir o nome da sua máquina:

Método 1 (Pelo Terminal): Abra o Prompt de Comando (CMD) e digite o comando hostname. O texto exibido logo abaixo é o nome oficial do seu computador.

Método 2 (Pelas Configurações): Pressione as teclas Windows + I, acesse Sistema e observe o nome exibido na parte superior da tela (ao lado do ícone do computador).

📋 PASSO A PASSO PARA REPARO DO SISTEMA
Etapa 1: Entrar na Tela Azul de Recuperação
Execute o comando abaixo para reiniciar o computador no menu avançado do Windows:

DOS
shutdown /r /o /t 00
O que este comando faz: Força o Windows a reiniciar direto no menu azul de recuperação (WinRE), ignorando a tela normal do sistema que estava travada.

Na tela azul, clique em: Solução de Problemas > Opções Avançadas > Configurações de Inicialização.

Clique no botão Reiniciar.

Na lista de opções, pressione a tecla 6 (ou F6) para selecionar Habilitar Modo de Segurança com Prompt de Comando.

*Etapa 2: Restaurar as Permissões do Disco C:*
No Prompt de Comando do Modo de Segurança, copie e cole os comandos abaixo, um por um, pressionando Enter após cada linha:

DOS
chcp 65001
O que este comando faz: Ajusta a codificação do terminal para UTF-8. Isso garante que o Windows reconheça corretamente nomes de pastas ou usuários que possuem acentos ou caracteres especiais.

DOS
takeown /f C:\ /a
O que este comando faz: Toma a posse ("propriedade") do disco C:\ para o grupo de Administradores. No Windows, você só consegue alterar permissões de uma pasta depois de se tornar o "dono" dela.

DOS
icacls C:\ /grant Administradores:(OI)(CI)F /c
O que este comando faz: Dá controle total ao grupo de Administradores. As siglas (OI)(CI) fazem essa permissão ser repassada automaticamente para todas as pastas e arquivos dentro do disco C:.

DOS
icacls C:\ /grant Todos:(OI)(CI)RX /c
O que este comando faz: Concede permissão de Leitura e Execução (RX) para os usuários comuns, permitindo que os programas do sistema voltem a abrir normalmente.

Etapa 3: Corrigir a Lixeira e Reiniciar
DOS
rd /s /q C:\$Recycle.Bin
O que este comando faz: Apaga a pasta invisível e corrompida da Lixeira no C:. O Windows recria essa pasta do zero e 100% funcional na próxima vez que ligar.

DOS
shutdown /r /t 00
O que este comando faz: Reinicia o computador imediatamente de volta para o modo normal com todos os acessos liberados.
