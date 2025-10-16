# Logbook 2 - Caracterização do CVE-2017-0144

## **Identificação**

-  **Vulnerabilidade SMBv1**: Falha crítica no protocolo Server Message Block versão 1 do Microsoft Windows [[1]](#ref1)[[2]](#ref2)
-  **Execução remota de código**: Permite atacantes executarem código arbitrário através de pacotes maliciosos crafted[[3]](#ref3)[[4]](#ref4)
-  **Sistemas afetados**: Windows 7, 8.1, 10, Server 2008/2012/2016, Vista SP2[[5]](#ref5)[[1]](#ref1)
-  **Protocolo de rede**: Exploração via porta 445 TCP do serviço SMB[[6]](#ref6)[[7]](#ref7)

## **Catalogação**

-  **Reporting NSA**: Descoberto pela NSA, usado como cyber-arma antes do vazamento pelos Shadow Brokers[[8]](#ref8)[[9]](#ref9)
-  **Disclosure público**: 17 março 2017 (CVE), 14 abril 2017 (leak Shadow Brokers)[[10]](#ref10)[[8]](#ref8)
-  **Gravidade CVSS**: Score 8.1-9.3 HIGH, EPSS 94.32% probabilidade de exploração[[3]](#ref3)[[1]](#ref1)
-  **Catalogação CISA**: Incluído no Known Exploited Vulnerabilities Catalog, ação requerida até agosto 2022[[2]](#ref2)[[3]](#ref3)

## **Exploit**

-  **EternalBlue NSA**: Exploit original desenvolvido pela NSA, vazado e disponibilizado publicamente[[7]](#ref7)[[8]](#ref8)
-  **Metasploit modules**: ms17_010_eternalblue, ms17_010_eternalblue_win8, smb_ms17_010 scanner disponíveis[[11]](#ref11)[[3]](#ref3)
-  **Automação completa**: Exploração totalmente automatizada, não requer interação do usuário[[12]](#ref12)[[11]](#ref11)
-  **Propagação worm**: Capacidade de auto-propagação lateral em redes através de SMBv1[[13]](#ref13)[[14]](#ref14)

## **Ataques**

-  **WannaCry 2017**: 300.000+ computadores infectados em 150 países, $4 bilhões em danos[[14]](#ref14)[[8]](#ref8)
-  **NotPetya 2017**: $1 bilhão em danos, 65 países afetados, infraestrutura crítica comprometida[[15]](#ref15)[[8]](#ref8)
-  **Ataques persistentes**: LemonDuck, TrickBot, Retefe continuam explorando a vulnerabilidade atualmente[[16]](#ref16)[[13]](#ref13)
-  **Infraestrutura crítica**: NHS, ferrovias, manufatura automotiva, sistemas governamentais severamente impactados[[15]](#ref15)[[14]](#ref14)

## **Correção/Contramedidas**

-  **Patch MS17-010**: Microsoft Security Bulletin publicado 14 março 2017, correção crítica obrigatória[[17]](#ref17)[[18]](#ref18)
-  **Desabilitar SMBv1**: `Set-SmbServerConfiguration -EnableSMB1Protocol $false` via PowerShell[[19]](#ref19)[[6]](#ref6)
-  **Verificação pós-patch**: Nessus plugin 97737, Trend Micro validation tool para confirmar correção[[20]](#ref20)[[21]](#ref21)
-  **Mitigação de rede**: Bloqueio porta 445, segmentação de rede, monitorização SMB ativa[[6]](#ref6)[[5]](#ref5)


## **Referências**

<a id="ref1"></a>[1]: https://www.incibe.es/en/incibe-cert/early-warning/vulnerabilities/cve-2017-0144  
<a id="ref2"></a>[2]: https://www.twingate.com/blog/tips/cve-2017-0144  
<a id="ref3"></a>[3]: https://www.cvedetails.com/cve/CVE-2017-0144/  
<a id="ref4"></a>[4]: https://nvd.nist.gov/vuln/detail/cve-2017-0144  
<a id="ref5"></a>[5]: https://www.clouddefense.ai/cve/2017/CVE-2017-0144  
<a id="ref6"></a>[6]: https://calcomsoftware.com/disable-hardening-smbv1/  
<a id="ref7"></a>[7]: https://www.sentinelone.com/blog/eternalblue-nsa-developed-exploit-just-wont-die/  
<a id="ref8"></a>[8]: https://en.wikipedia.org/wiki/EternalBlue  
<a id="ref9"></a>[9]: https://www.rapid7.com/blog/post/2017/04/18/the-shadow-brokers-leaked-exploits-faq/  
<a id="ref10"></a>[10]: https://research.checkpoint.com/2017/eternalblue-everything-know/  
<a id="ref11"></a>[11]: https://0xma.github.io/hacking/eternal_blue_metasploit.html  
<a id="ref12"></a>[12]: https://github.com/EEsshq/CVE-2017-0144---EtneralBlue-MS17-010-Remote-Code-Execution  
<a id="ref13"></a>[13]: https://cyberscoop.com/retefe-eternal-blue-nsa-proofpoint/  
<a id="ref14"></a>[14]: https://en.wikipedia.org/wiki/WannaCry_ransomware_attack  
<a id="ref15"></a>[15]: https://www.wired.com/story/nsa-zero-day-symantec-buckeye-china/  
<a id="ref16"></a>[16]: https://socprime.com/blog/detect-lemonduck-malware-attacks/  
<a id="ref17"></a>[17]: https://learn.microsoft.com/en-us/security-updates/securitybulletins/2017/ms17-010  
<a id="ref18"></a>[18]: https://success.trendmicro.com/en-US/solution/KA-0008859  
<a id="ref19"></a>[19]: https://horizon3.ai/fix-actions/cve-2017-0144/  
<a id="ref20"></a>[20]: https://community.avg.com/t/multiple-pop-ups-with-threat-secured-smb-cve-2017-0144/246682  
<a id="ref21"></a>[21]: https://www.tenable.com/plugins/nessus/97833  
<a id="ref22"></a>[22]: https://www.redlings.com/en/guide/cvss-score  
<a id="ref23"></a>[23]: https://www.tenable.com/cve/CVE-2017-0144  
<a id="ref24"></a>[24]: https://www.youtube.com/watch?v=O-dF504aB6U  
<a id="ref25"></a>[25]: https://www.cve.org/CVERecord?id=CVE-2017-0144  
<a id="ref26"></a>[26]: https://access.redhat.com/security/cve/CVE-2017-0144  
<a id="ref27"></a>[27]: https://nvd.nist.gov/vuln/detail/CVE-2017-0144/change-record?changeRecordedOn=03%2F17%2F2017T15%3A36%3A16.853-0400  
<a id="ref28"></a>[28]: https://msrc.microsoft.com/update-guide/vulnerability/CVE-2017-0144  
<a id="ref29"></a>[29]: https://otx.alienvault.com/indicator/cve/CVE-2017-0144  
<a id="ref30"></a>[30]: https://learn.microsoft.com/pt-pt/security-updates/securitybulletins/2017/ms17-010  
<a id="ref31"></a>[31]: https://www.rapid7.com/db/vulnerabilities/msft-cve-2017-0144/  
<a id="ref32"></a>[32]: https://www.isaca.org/resources/isaca-journal/issues/2017/volume-5/exposing-the-fallacies-of-security-by-obscurity-full-disclosure  
<a id="ref33"></a>[33]: https://www.avast.com/c-eternalblue  
<a id="ref34"></a>[34]: https://www.halcyon.ai/faqs/what-is-eternalblue  
