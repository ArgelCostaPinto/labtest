Laboratório de teste utilizando Medusa

Teste 1 — Ataque FTP com Medusa

Na primeira atividade foi usado o Medusa para forçar logins no serviço FTP do alvo (192.168.56.101). A ideia foi simples: tentar combinações de usuários e senhas até encontrar uma que funcionasse.

Comando usado:
medusa -h 192.168.56.101 -U users.txt -P pass.txt -M ftp -t 6

O Medusa rodou várias tentativas em paralelo contra o serviço FTP. No fim ele reportou sucesso com as credenciais Login:msfadmin,Senha:msfadmin. Para validar:

ftp 192.168.56.101
  usuário: msfadmin
  senha: msfadmin

O prompt aceitou o login — ou seja, a combinação encontrada realmente permitiu acesso ao serviço FTP. Foi um exemplo prático de como contas com credenciais previsíveis podem ser exploradas rápido por ferramentas automatizadas.

Teste 2 — Brute-force em formulário web (DVWA)

Aqui o alvo foi a página de login do DVWA (/dvwa/login.php). Foi testado um login manual primeiro (ex.: Argel / 123) e recebemos Login Failed. Mas, ao inspecionar a requisição no DevTools, vimos os campos do formulário: username, password e Login.

Foi montado um ataque direcionado ao POST do formulário com Medusa, especificando a página, o corpo do formulário e a string que indica falha:

Comando usado:
  medusa -h 192.168.56.101 -U users.txt -P pass.txt -M http \
    -m PAGE:'/dvwa/login.php' \
    -m FORM:'username=^USER^&password=^PASS^&login=Login' \
    -m 'FAIL=login failed' -t 6

O Medusa varreu combinações e encontrou admin:password. Novamente, foi validado que o acesso era real. Esse teste mostra que, em aplicações web que não protegem o formulário contra tentativas automáticas, ataques de força bruta conseguem identificar credenciais válidas.

Teste 3 — Enumeração SMB + Password spraying (fluxo em cadeia)

Nesse exercício primeiro foi feita um enumeração do serviço SMB para coletar informações úteis (nomes, workgroup, possíveis usuários), depois esses dados foram usados para montar wordlists e executar um password spray.

Enumeração:
  enum4linux -a 192.168.56.101 | tee enum4_output.txt

O output trouxe nomes e comentários que nos ajudaram a criar listas de usuários e senhas mais direcionadas.

Ataque com Medusa (SMB):
  medusa -h 192.168.56.101 -U smb_user.txt -P senhas_spray.txt -M smbnt -t 2 -T 50

O Medusa retornou Login:msfadmin,Senha:msfadmin como credenciais válidas.

Validação com smbclient:
  smbclient -L //192.168.56.101 -U msfadmin
    senha: msfadmin
    
O acesso foi concedido, confirmando que a credencial funcionava no serviço SMB.

Os três testes demonstraram de forma prática como serviços expostos com credenciais padrão ou fracas e sem proteções básicas são rapidamente comprometidos por ferramentas automatizadas, ao combinar enumeração cuidadosa e ataques em cadeia foi possivel validar acessos tanto em FTP quanto em formulários web e SMB, o que deixa claro que é essencial remover ou alterar contas e senhas padrão, reduzir a exposição de serviços desnecessários, manter os sistemas atualizados e revisar permissões e compartilhamentos para limitar o que realmente precisa ficar acessível, além disso, realizar varreduras e testes autorizados periodicamente ajuda a identificar e corrigir pontos fracos antes que sejam explorados por agentes maliciosos.
