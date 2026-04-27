#Detecção de Descoberta de Rede
# Detecção de Descoberta de Rede
---

## O que é descoberta de rede?

Antes de atacar, todo atacante precisa entender o alvo. Essa fase é chamada de descoberta de rede — o atacante tenta mapear o que existe na rede: quais IPs estão ativos, quais portas estão abertas, quais serviços estão rodando e se algum deles tem vulnerabilidade conhecida.

O problema é que defensores fazem a mesma coisa. Equipes de segurança varrem a própria rede regularmente para inventariar ativos, fechar portas desnecessárias e corrigir vulnerabilidades. Mecanismos de busca e pesquisadores também fazem varreduras legítimas na internet.

O trabalho do analista SOC é diferenciar uma varredura legítima de uma maliciosa.

---

## Varredura externa vs varredura interna

A primeira coisa a observar em qualquer varredura é: de onde ela veio?

### Varredura externa
O IP de origem é externo e o destino é um ativo da organização.

Isso indica que o atacante **ainda não entrou na rede** — está na fase de reconhecimento inicial, tentando encontrar uma brecha para obter acesso.

- Severidade: **baixa**
- Resposta: bloquear o IP no firewall de perímetro
- Limitação: o atacante pode mascarar o IP e tentar novamente

### Varredura interna
O IP de origem e o IP de destino são ambos endereços privados dentro da rede.

Isso indica que o atacante **já está dentro da rede** e está se preparando para movimento lateral — procurando outras máquinas para comprometer.

- Severidade: **alta**
- Resposta: bloquear o IP no firewall não resolve — é necessário iniciar resposta a incidentes, investigar o host de origem e identificar a causa raiz

---

## Varredura horizontal vs varredura vertical

Além da origem, o comportamento da varredura também revela a intenção do atacante.

### Varredura horizontal
**Mesmo IP de origem → mesma porta → vários IPs de destino**

O atacante quer saber quais máquinas da rede têm aquela porta aberta. Geralmente feito quando o atacante pretende explorar um serviço específico em qualquer máquina vulnerável.

Exemplo real: o ransomware WannaCry varreu redes inteiras procurando máquinas com a porta 445 (SMB) aberta para se espalhar.

Como identificar nos logs:

```
src_ip: sempre o mesmo
dst_port: sempre a mesma
dst_ip: varia em cada evento
```
### Varredura vertical
**Mesmo IP de origem → mesmo IP de destino → várias portas**

O atacante está focado em uma máquina específica e quer mapear todos os serviços que ela expõe. Geralmente feito quando o atacante identificou um alvo de alto valor e quer encontrar a melhor forma de explorá-lo.

Exemplo real: atacante que identifica um servidor web exposto e varre todas as portas para encontrar serviços mal configurados ou vulneráveis.

Como identificar nos logs:

```
src_ip: sempre o mesmo
dst_ip: sempre o mesmo
dst_port: varia em cada evento
```
---

## Técnicas de varredura

### Ping Scan (ICMP)
Envia um pacote ICMP para o host. Se o host responder, está online.

- Vantagem: simples e rápido
- Desvantagem: fácil de bloquear — muitas organizações bloqueiam ICMP no perímetro

### TCP SYN Scan
Aproveita o handshake TCP (SYN → SYN-ACK → ACK). O scanner envia um SYN e espera o SYN-ACK. Se receber, a porta está aberta.

- Vantagem: furtivo — se mistura com tráfego normal
- Desvantagem: mais difícil de detectar justamente por isso

### UDP Scan
Envia um pacote UDP vazio. Se a porta estiver fechada, o host responde com ICMP "porta inacessível". Se não houver resposta, a porta pode estar aberta.

- Vantagem: identifica serviços UDP (DNS, SNMP, DHCP)
- Desvantagem: lento e pouco confiável — depende de timeout para concluir

---

## Como o analista SOC diferencia varreduras legítimas de maliciosas

| Técnica | Como funciona |
|---|---|
| Whitelist | Scanners internos e externos conhecidos são adicionados a uma lista de permissões — nenhum alerta é gerado para essas fontes |
| Threat Intelligence | Integrar feeds de IPs maliciosos conhecidos — alertar apenas quando a varredura vem de uma fonte suspeita |
| Aumento de severidade | Usar Threat Intelligence para elevar a prioridade de alertas de fontes suspeitas, ao invés de apenas bloquear |
| Casos de uso genéricos | Criar regras que alertam sobre comportamento de varredura independente da fonte — captura o que a Threat Intelligence deixa passar |

---

## Resumo — o que observar nos logs

| Padrão | Tipo de varredura | Significado |
|---|---|---|
| IP externo → vários IPs internos, mesma porta | Horizontal externa | Reconhecimento inicial — atacante mapeando superfície de ataque |
| IP externo → mesmo IP interno, várias portas | Vertical externa | Atacante focado em um ativo específico exposto |
| IP interno → vários IPs internos, mesma porta | Horizontal interna | Atacante já dentro da rede — alto risco, possível movimento lateral |
| IP interno → mesmo IP interno, várias portas | Vertical interna | Atacante mapeando host interno específico — alto risco |

---
*TryHackMe | Network Security — Network Discovery Detection*
