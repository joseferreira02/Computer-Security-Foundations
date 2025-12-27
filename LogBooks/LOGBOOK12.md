# LOGBOOK 11 - Sniffing and Spoofing Lab

## Task 1.1.A: Os containers hostA e hostB não enviam pacotes por defeito. Para exemplificar esta tarefa, deverá enviar pacotes de dentro de cada container. Demonstre e explique as várias camadas dos pacotes sniffed.

After using the command `dockup` and verifying that the containers where working as expected, we created `sniffer.py`.

Following the guide, we changed the interface in the example with the Host VM one (SEED).

### Sniffer.py
```
#!/usr/bin/env python3
from scapy.all import *

def print_pkt(pkt):
    pkt.show()

pkt = sniff(iface='br-49fee2210386', filter='icmp', prn=print_pkt)
```

### Results

![Ping from HostB](../images/logbook12/lab12_task1.1_pinghostb.png)

![Scapy Ping Sniff](../images/logbook12/lab12_task1.1_scapyping.png)

![Sniffer Error](../images/logbook12/lab12_task1_permissionerror.png)
## Task 1.1B: Deve descobrir qual a sintaxe necessária para os filtros BPF e quais comandos a executar para enviar os pacotes desejados.

As the code from the guide already had the icmp filter in the line `pkt = sniff(iface='br-49fee2210386', filter='icmp', prn=print_pkt)`, we got a tip on how to configure



![ICMP Filter](../images/logbook12/task1.1b_icmpfilter.png)


`pkt = sniff(iface='br-49fee2210386', filter='tcp and src host 10.9.0.5 and dst port 23', prn=print_pkt)`
### Telnet Command
![TCP and Host](../images/logbook12/task1.1b_tcpfilter_telnet.png)

### Result on sniffer
![TCP and Host](../images/logbook12/task1.1b_tcpfilter.png)
## Task 1.2: Garanta que corre o wireshark com permissões de root.
## Task 1.3: Escolha algum IP externo, como por exemplo 8.8.8.8.
## Task 1.4: Primeiro descubra como sniffar e filtrar ICMP echo requests, lembrando-se da Task 1.1. Num dos containers, pode enviar ICMP echo requests com o comando ping. Depois envie o pacote de resposta adaptando as soluções das Task 1.2 e 1.3.