# prompts.md — Global Solution 2026.2 · CitizenTech

**CitizenTech:** Busão Inteligente
**Equipe:** Ponto Cego — equipe 10
**Integrantes:** Felipe Balbino Murad (RM562347) · Lucas Mesquita Massoni (RM561686)
**Disciplinas:** Network Architect Solutions · Data Science & Statistical Computing ·
Database Design · Domain Driven Design (Java) · Dynamic Programming ·
Agile Methodology With Squad Framework · AR-VR Modelling And Simulation

---

## 1. Por que este arquivo existe

O briefing da GS autoriza IA como copiloto com duas condições explícitas: os prompts
ficam registrados e cada linha entregue precisa ser explicável por qualquer integrante
da equipe. Este arquivo cumpre a primeira condição. A segunda é responsabilidade
de cada um de nós: nada entra no projeto sem que saibamos defender a decisão.

Regra da equipe: **se não conseguimos explicar sem consultar a IA, o trecho não está pronto.**

---

## 2. Uso de IA neste projeto

O combinado da equipe e o briefing da GS são explícitos: IA como copiloto, permitida para
aprender e revisar, com os prompts registrados em arquivo e cada linha explicável. Este
arquivo cumpre o registro. A explicação de cada trecho está na seção 4, e qualquer
integrante consegue defendê-la sem consultar a IA.

### 2.1 IA usada para aprender

- Entender comportamento de protocolo: por que o TTL cai de 128 para 127, por que a ACL
  gera "destination host unreachable" em vez de "request timed out", por que a ordem das
  regras de uma ACL estendida altera o resultado.
- Entender a diferença entre SSH habilitado e Telnet desabilitado, e por que
  `transport input ssh` é a linha que cumpre o requisito.
- Entender por que a VLAN nativa deve ser dedicada, e o que é VLAN hopping.
- Entender por que o Wi-Fi pode ser público e usar WPA2 ao mesmo tempo.

Em todos esses casos perguntamos o motivo, não a resposta pronta. O que ficou no projeto
foi o entendimento, e ele está escrito na seção 8 do relatório.

### 2.2 IA usada para revisar

- Revisão do plano de endereçamento: conferir a divisão VLSM e se cada máscara comporta
  o número de dispositivos do setor.
- Auditoria do enunciado contra o projeto: localizar requisito não cumprido. Foi assim que
  descobrimos que o R-ISP não tinha SSH configurado.
- Revisão do PDF antes da entrega: apontar print ausente, tabela quebrada e conteúdo
  repetido.
- Depuração de mensagem de erro do simulador: módulo wireless recusado, sintaxe do
  `crypto key generate rsa`, comando digitado no dispositivo errado.

Nesses casos a IA apontou o problema; a correção foi feita e testada por nós no Packet
Tracer, e é comprovada pelos prints.

### 2.3 Apoio de redação — declarado

O texto corrido do relatório foi redigido com apoio de IA, a partir de fatos, valores e
saídas de comando que já tínhamos produzido no simulador. Declaramos isso em vez de omitir.

Nenhuma decisão técnica nasceu desse apoio: a topologia, a divisão das sub-redes, as regras
da ACL, as exclusões de DHCP e os parâmetros do Wi-Fi foram definidos e validados por nós.
A IA organizou em texto o que a equipe já havia decidido e testado.

### 2.4 O que a IA não fez

- Não executou nada no Packet Tracer. Toda configuração foi digitada e testada pela equipe.
- Não produziu evidência. Todos os prints vêm de execução real no simulador.
- Não definiu os valores da rede. Faixas, VLANs, exclusões e regras foram calculadas por nós.
- Não fez o experimento de Wi-Fi, os testes de ping, nem as capturas de tela.

### 2.5 Regra de dados

Nenhum dado pessoal real foi usado. Endereços IP, MACs e números de porta são do simulador.

---

## 2.6 Como cada registro se classifica

| Registro | Finalidade | O que fizemos |
|---|---|---|
| P-01 · Cálculo VLSM | Aprender | Conferimos a matemática à mão e validamos com `show ip dhcp pool` |
| P-02 · Auditoria do enunciado | Revisar | Corrigimos o R-ISP, que estava sem SSH |
| P-03 · Erro vindo do gateway | Aprender | Usamos a explicação na seção 7.5 do PDF |
| P-04 · Wi-Fi público com WPA2 | Aprender | Adotamos o argumento e acrescentamos o detalhe do totem |
| P-05 · Adaptador wireless | Revisar | Trocamos o módulo e refizemos a associação |
| P-06 · SSH × Telnet | Aprender + Revisar | Conferimos `line vty` e trocamos senha por `secret` |
| P-07 · Erro do `crypto key` | Revisar | Corrigimos a sintaxe e geramos a chave |
| P-08 · Auditoria do PDF | Revisar | Corrigimos os quatro pontos críticos |
| P-09 · Cobertura dos testes | Revisar | Acrescentamos a seção 6.2 ao relatório |
| P-10 · Registro dos erros da IA | Revisar | Cada erro foi detectado ao testar no simulador |

---

## 3. Registro de prompts — Network Architect Solutions

### P-01 · Cálculo do plano de endereçamento

**Contexto enviado:** faixa 192.168.10.0/24 (equipe 10), três setores com perfis
distintos de consumo de endereço, necessidade de reserva para expansão de sensores.

**O que pedimos:** dividir o /24 por VLSM em três sub-redes proporcionais ao setor,
indicando sub-rede, máscara, gateway e faixa DHCP de cada uma, com o bloco livre
explicitado.

**O que voltou:** /25 para Cidadão (192.168.10.0/25), /26 para Equipe
(192.168.10.128/26), /27 para Servidores (192.168.10.192/27) e 192.168.10.224/27
livre para expansão.

**O que fizemos:** conferimos a matemática à mão. /25 entrega 126 hosts úteis, /26
entrega 62, /27 entrega 30 — coerente com o perfil de cada setor. Adotamos.

**Verificação:** `show ip dhcp pool` retornou Total addresses 126, 62 e 30,
exatamente os valores calculados. O plano está correto na prática.

---

### P-02 · Auditoria do enunciado contra o projeto pronto

**Contexto enviado:** os quatro blocos de requisitos do professor (endereçamento e
VLANs, serviços, segurança, testes) e o estado do projeto.

**O que pedimos:** conferir item por item o que já estava cumprido, o que faltava e
o que estava ambíguo.

**O que voltou:** uma lista com 19 itens, dos quais 16 cumpridos, 1 incompleto
(SSH no R-ISP) e 2 pendentes de montagem.

**O que fizemos:** o item do R-ISP era uma falha real e não tínhamos percebido. O
enunciado diz "roteador só por SSH" e nós só havíamos configurado o R1. Corrigimos
o R-ISP.

**O que descartamos:** a IA sugeriu verificar também os switches. O enunciado fala
em "roteador", não em switch. Não alteramos os switches por causa disso.

**Verificação:** `show ip ssh` no R1 e no R-ISP retornando `SSH Enabled - version 2.0`.

---

### P-03 · Por que a mensagem de erro veio do gateway

**Contexto enviado:** saída de `ping 192.168.10.130` do TOTEM-CIDADAO retornando
`Reply from 192.168.10.1: Destination host unreachable` nos quatro pacotes.

**O que pedimos:** explicar por que a resposta vinha do gateway e não do destino,
e por que isso era diferente de um host desligado.

**O que voltou:** a explicação de que o próprio roteador gera a mensagem ICMP de
destino inalcançável ao descartar o pacote pela ACL. Host desligado produziria
"Request timed out", sem remetente.

**O que fizemos:** usamos a explicação na seção 7.5 do PDF, porque ela distingue
bloqueio intencional de indisponibilidade — que é exatamente o que o teste precisa
provar.

**Verificação:** confirmado com `show access-lists 100`, onde o `deny` para a
VLAN 10 acumulou 8 correspondências.

---

### P-04 · Justificativa do Wi-Fi público com WPA2

**Contexto enviado:** o enunciado pede, na mesma frase, "Wi-Fi para o público" e
"Wi-Fi com WPA2". Isso parecia uma contradição.

**O que pedimos:** uma forma honesta de justificar os dois requisitos convivendo.

**O que voltou:** a distinção entre acesso livre (sem cadastro nem autenticação
individual) e transmissão cifrada (WPA2-PSK com AES). A rede é pública no primeiro
sentido e protegida no segundo.

**O que fizemos:** adotamos o argumento e acrescentamos o detalhe operacional que
faltava: a chave é divulgada no próprio totem do terminal, o que mantém o acesso
aberto na prática sem abrir mão da cifragem.

**Verificação:** configuração do AP-CIDADAO — WPA2-PSK, AES, canal 6 fixo.

---

### P-05 · Depuração do adaptador wireless no Packet Tracer

**Contexto enviado:** mensagem de erro `A WMP300N or WPC300N wireless interface is
required to connect` ao abrir o PC Wireless no LAPTOP-VISITANTE, com o módulo
PT-LAPTOP-NM-1W instalado.

**O que pedimos:** por que o módulo instalado não era aceito, se ele aparecia na
biblioteca do simulador.

**O que voltou:** o aplicativo PC Wireless exige o WPC300N (ou WMP300N) e recusa o
PT-LAPTOP-NM-1W, apesar de ele estar disponível na lista. É uma inconsistência do
próprio simulador.

**O que fizemos:** desligamos o laptop, trocamos o módulo pelo WPC300N, religamos e
refizemos a associação. A interface passou de FastEthernet0 para Wireless0.

**Verificação:** `show ip dhcp binding` passou a registrar 192.168.10.12 com o MAC
0030.F259.C2BE, prefixo de interface wireless.

---

### P-06 · Diferença entre SSH habilitado e Telnet desabilitado

**Contexto enviado:** `show ip ssh` retornando `SSH Enabled - version 2.0` e a
dúvida sobre o requisito "roteador só por SSH".

**O que pedimos:** verificar se SSH habilitado já cumpria o requisito.

**O que voltou:** não. O padrão do IOS aceita Telnet e SSH simultaneamente na vty.
Só `transport input ssh` bloqueia o Telnet.

**O que fizemos:** conferimos o bloco `line vty` e confirmamos `transport input ssh`
e `login local`. Também trocamos qualquer senha em texto claro por `secret`.

**Verificação:** `show running-config | begin line vty` e
`show running-config | include username` mostrando `secret 5`.

**Este foi o prompt mais valioso do projeto.** Sem ele, o item de segurança estaria
aprovado por engano.

---

### P-07 · Erro de sintaxe na geração da chave RSA

**Contexto enviado:** `crypto key generate rsa modulus 1024` retornando
`% Invalid input detected at '^' marker`.

**O que pedimos:** a sintaxe correta para o Packet Tracer.

**O que voltou:** sem o parâmetro `modulus` na linha. O tamanho da chave vem por
prompt interativo.

**O que fizemos:** rodamos `crypto key generate rsa` e digitamos 1024 no prompt.

**Verificação:** `show ip ssh` passou a responder no R-ISP.

---

### P-08 · Auditoria do PDF antes da entrega

**Contexto enviado:** o PDF já montado com os prints colados.

**O que pedimos:** revisão crítica contra o enunciado, sem elogio, apontando o que
estava faltando ou errado.

**O que voltou:** 15 apontamentos, entre eles quatro críticos: capa com CitizenTech
e Equipe invertidos, seção de topologia sem print, e prints ausentes nas seções de
Wi-Fi, NAT e contadores da ACL.

**O que fizemos:** conferimos cada apontamento no arquivo, corrigimos os quatro
críticos e recortamos o `show running-config` que se repetia idêntico três vezes.

**Verificação:** releitura do PDF final conferindo cada marcador de print preenchido.

---

### P-09 · Conferência de cobertura dos testes

**Contexto enviado:** o conjunto de saídas coletadas até aquele momento.

**O que pedimos:** verificar se faltava algum dos testes obrigatórios.

**O que voltou:** todos os cinco testes exigidos estavam cobertos, mas faltava a
evidência de SSH, que o briefing menciona fora da lista de testes.

**O que fizemos:** acrescentamos a seção 6.2 e capturamos `show ip ssh`, `line vty`
e as linhas de `username` e `enable secret`.

**Verificação:** seção 6.2 do PDF.

---

### P-10 · Registro do que a IA errou

Documentado de propósito. O briefing pede uso crítico, não uso cego.

**Erro 1 — Seção "Wireless" inexistente.** Foi sugerido abrir Config → Wireless
para configurar o AP. O AP-CIDADAO é um AccessPoint-PT e não tem essa seção: os
parâmetros ficam em Config → INTERFACE → Port 1.

**Erro 2 — Instrução para desligar o DHCP do AP.** Foi sugerido desativar o servidor
DHCP do access point. Esse modelo não tem servidor DHCP — é bridge de camada 2. A
instrução não se aplicava e foi descartada.

**Erro 3 — Comando no dispositivo errado.** `show ip nat translations` foi digitado
no Command Prompt do PC e retornou `Invalid Command`. É comando de IOS, só existe
no roteador.

**Erro 4 — Marcador de print com escopo ambíguo.** Um marcador pedia "switchport mode
trunk e switchport access vlan" num único print, misturando dois equipamentos com
funções diferentes. Foi dividido em dois prints, um para o SW-CORE e outro para os
switches de setor.

**Lição registrada:** toda orientação de configuração foi conferida no equipamento
antes de virar decisão. As três primeiras falhas foram detectadas justamente porque
testamos em vez de aceitar.

---

## 4. Explicação linha por linha — o que a banca pode perguntar

### ACL 100 (aplicada inbound na Gi0/0/0.20)

    permit udp any any eq bootps
    deny ip 192.168.10.0 0.0.0.127 192.168.10.128 0.0.0.63
    permit tcp 192.168.10.0 0.0.0.127 host 192.168.10.194 eq www
    permit tcp 192.168.10.0 0.0.0.127 host 192.168.10.194 eq 443
    permit udp 192.168.10.0 0.0.0.127 host 192.168.10.194 eq domain
    permit tcp 192.168.10.0 0.0.0.127 host 192.168.10.194 eq domain
    deny ip 192.168.10.0 0.0.0.127 192.168.10.192 0.0.0.31
    permit ip 192.168.10.0 0.0.0.127 any

**Ordem das regras.** O servidor (192.168.10.194) pertence ao bloco da VLAN 30, que
a penúltima linha bloqueia. Se o bloqueio à VLAN 30 viesse antes das permissões ao
servidor, a página não abriria e o requisito de HTTP falharia por causa da própria
ACL. Por isso as permissões específicas vêm primeiro.

**`permit udp any any eq bootps`.** Libera DHCP antes de qualquer bloqueio. Sem ela,
o cliente da VLAN 20 perderia o endereço na primeira renovação de concessão.

**`deny ip ... 0.0.0.63`.** O bloco Equipe é 192.168.10.128/26; a wildcard 0.0.0.63
cobre exatamente esse intervalo.

**`permit ... eq www` e `eq 443`.** Libera HTTP e HTTPS ao servidor. A 443 está ali
prevendo a migração da página para TLS.

**`permit udp ... eq domain` e `permit tcp ... eq domain`.** DNS em UDP para consulta
comum e em TCP para transferência de zona.

**`deny ip ... 0.0.0.31`.** Bloqueia o restante da VLAN 30, incluindo ICMP e acesso
aos sensores. Como o servidor já foi liberado acima, ele continua acessível.

**`permit ip ... any`.** Libera o tráfego remanescente — na prática, a internet.

**Por que o ping do cidadão ao servidor falha.** ICMP não está liberado. O teste de
HTTP é feito no navegador, que é funcionalmente correto: um ping respondendo não
provaria que a página abre.

### NAT/PAT

    ip nat inside source list 1 interface GigabitEthernet0/0/1 overload

Todos os endereços permitidos pela ACL 1 (o /24 inteiro) saem pelo IP da WAN,
198.51.100.2, com tradução de porta. O `overload` é o que transforma NAT em PAT.

### BGP de exclusão do DHCP

    ip dhcp excluded-address 192.168.10.192 192.168.10.209

Cobre o gateway .193 e o servidor .194. Foi feita em um único intervalo em vez de
linhas separadas, o que também reserva .195 a .209 para futuros sensores fixos.

---

## 5. Outras disciplinas

O registro segue o mesmo formato. Preencher conforme o projeto avança:

- **Data Science** — prompts sobre escolha de variáveis-alvo, interpretação de MAE,
  RMSE, R², recall e F1; justificativa do split treino/teste; leitura da matriz de
  confusão.
- **Database Design** — prompts sobre cardinalidades, normalização, escolha de PK e
  as três consultas exigidas (INNER JOIN, GROUP BY e LEFT OUTER JOIN).
- **Domain Driven Design** — prompts sobre modelagem de domínio, camadas DAO/BO,
  tratamento de exceções e deploy.
- **Dynamic Programming** — prompts sobre escolha entre memoização, tabulação e
  grafos, com análise de complexidade.
- **Agile** — prompts sobre priorização MoSCoW, histórias de usuário e estimativa
  por Planning Poker.

Em todas: registrar o que foi aceito, o que foi rejeitado e por quê.

---

## 6. Checklist antes de considerar o projeto entregue

- [x] prompts.md na raiz do repositório
- [ ] Ficha da CitizenTech no início do README
- [ ] Um integrante de cada disciplina sabe explicar qualquer trecho da sua entrega
- [ ] Nenhum dado pessoal real no projeto
- [ ] Nenhum link externo como entrega (PDF único quando exigido)
- [ ] Commits feitos antes do prazo de 19/11, 23h55
