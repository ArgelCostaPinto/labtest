Laboratório de Testes de Invasão com Medusa
Este repositório documenta uma série de testes de penetração realizados com a ferramenta Medusa para demonstrar vulnerabilidades comuns em diferentes serviços. Os testes foram focados em ataques de força bruta e enumeração contra FTP, formulários web e SMB.

Teste 1 — Ataque de Força Bruta em FTP
Nesta atividade, o objetivo foi forçar o login no serviço FTP do alvo (192.168.56.101) utilizando uma combinação de listas de usuários e senhas.

Comando Utilizado
bash
medusa -h 192.168.56.101 -U users.txt -P pass.txt -M ftp -t 6
Resultados e Validação
O Medusa executou múltiplas tentativas em paralelo e reportou sucesso ao encontrar as credenciais:

Login: msfadmin
Senha: msfadmin
A validação foi feita acessando o serviço diretamente, o que confirmou o acesso:

bash
ftp 192.168.56.101
# Usuário: msfadmin
# Senha: msfadmin
Conclusão do Teste: Este cenário demonstra como contas com credenciais previsíveis ou padrão podem ser rapidamente exploradas por ferramentas automatizadas.

Teste 2 — Força Bruta em Formulário Web (DVWA)
O alvo foi a página de login da aplicação web DVWA (/dvwa/login.php). Após uma tentativa de login manual falhar, a requisição POST foi inspecionada para identificar os campos do formulário (username, password, Login).

Comando Utilizado
O ataque foi direcionado ao endpoint do formulário, especificando o formato dos dados e a mensagem de falha.

bash
medusa -h 192.168.56.101 -U users.txt -P pass.txt -M http \
-m PAGE:'/dvwa/login.php' \
-m FORM:'username=^USER^&password=^PASS^&login=Login' \
-m 'FAIL=login failed' -t 6
Resultados e Validação
A ferramenta identificou as credenciais válidas:

Login: admin
Senha: password
O acesso foi validado manualmente na página de login.

Conclusão do Teste: Aplicações web que não implementam proteções contra automação em seus formulários (como CAPTCHA ou bloqueio de IP) são altamente vulneráveis a ataques de força bruta.

Teste 3 — Enumeração SMB e Password Spraying
Este exercício foi realizado em duas fases: primeiro, uma enumeração do serviço SMB para coletar informações e, em seguida, um ataque de password spraying usando os dados coletados.

Fase 1: Enumeração
O enum4linux foi usado para extrair nomes de usuários, workgroup e outros detalhes do alvo.

bash
enum4linux -a 192.168.56.101 | tee enum4_output.txt
Fase 2: Ataque com Medusa (SMB)
Com base nos dados da enumeração, foram criadas listas de usuários (smb_user.txt) e senhas (senhas_spray.txt) para um ataque mais direcionado.

bash
medusa -h 192.168.56.101 -U smb_user.txt -P senhas_spray.txt -M smbnt -t 2 -T 50
Resultados e Validação
O Medusa encontrou as seguintes credenciais válidas para o serviço SMB:

Login: msfadmin
Senha: msfadmin
A validação foi feita com o smbclient, que confirmou o acesso aos compartilhamentos.

bash
smbclient -L //192.168.56.101 -U msfadmin
# Senha: msfadmin
Conclusão do Teste: A combinação de enumeração e ataques em cadeia é extremamente eficaz. Coletar informações primeiro permite criar ataques mais precisos e com maior chance de sucesso.
