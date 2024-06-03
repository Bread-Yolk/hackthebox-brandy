# Ore
> Write-up author: jon-brandy

![image](https://github.com/jon-brandy/hackthebox/assets/70703371/008e35c8-a398-44a2-a5d1-b81a7a1e4ada)

## Lessons Learned:
- Reviewing Grafana and Catscale Output artifacts.
- Analyzing xmrig process.
- Hunting the Threat Actor's IP by reviewing UNIX auth log.

## SCENARIO:
<p align="justify">One of our technical partners are currently managing our AWS infrastructure. We requested the deployment of some technology into the cloud. The solution proposed was an EC2 instance hosting the Grafana application. Not too long after the EC2 was deployed the CPU usage ended up sitting at a continuous 98%+ for a process named "xmrig". Important Information Our organisation's office public facing IP is 86.5.206.121, upon the deployment of the application we carried out some basic vulnerability testing and maintenance.</p>

## STEPS:
1. In this case, we're tasked to investigate an anomaly that happened in Forela's AWS infrastructure. The infra itself is managed by a technical partner of Forela.
2. It is known that the CPU usage of the deployed EC2 instance unexpectedly soared to a constant of 98%+ due to a process named "XMRIG". The EC2 instance was used to host Grafana application.
3. As a cyber forensic, wr're tasked to investigate what "XMRIG" is due to our understanding about cloud infrastructure, EC2 instances, and system processes.

> 1ST QUESTION --> ANS: CVE-2021-43798

![image](https://github.com/jon-brandy/hackthebox/assets/70703371/a83b0a5f-0659-41af-99fd-8ce47e5d75fd)


3. Upon reviewing the Grafana artifacts, the Grafana's version used is shown at the `VERSION` file.

![image](https://github.com/jon-brandy/hackthebox/assets/70703371/d01dac73-5803-4e74-b34e-ccb81cc30528)


4. Searching on the internet for CVE related to the version, we found these results.

> RESULTS

![image](https://github.com/jon-brandy/hackthebox/assets/70703371/0784d0cb-81c6-447f-a43e-7067525a429d)


5. Reviewing each of them, drop us to a conclusion of the related CVE to this version should be --> `CVE-2021-43798`.

![image](https://github.com/jon-brandy/hackthebox/assets/70703371/6b259768-99c3-4edb-9bdc-2b3d0a195d5b)


> 2ND QUESTION --> ANS: 195.80.150.137, 

![image](https://github.com/jon-brandy/hackthebox/assets/70703371/215bb639-0ccc-42b6-a0b7-6fd5b60b9207)


6. Now let's review the catscale output.

![image](https://github.com/jon-brandy/hackthebox/assets/70703371/817bd491-6db8-41e8-9dea-9fa25b4b8dbf)


#### NOTES:

```
LinuxCatScale is a bash script that uses live of the land tools to collect extensive
data from Linux based hosts.

The data provided can help DFIR professinoals in triaging and scoping incidents.
The output from LinuxCatScale typically includes detailed reports on vulnerabilities,
configuration issues, and an overall security score to help administrators quickly understand
and address security weaknesses.
```

7. Navigating to the `/Logs` directory, we found 2 .gzip files. The var log should be our interest. Let's extract it.

![image](https://github.com/jon-brandy/hackthebox/assets/70703371/cf0a2448-659e-4137-92e2-d693054af189)


> RESULT

![image](https://github.com/jon-brandy/hackthebox/assets/70703371/08958846-c73a-4fa3-b340-035f6b63617b)


8. Our investigation shall start at the UNIX auth log. Based from the scenario, we know that IP 86.5.206.121 is a legitimate IP. It's Forela's public IP.
9. Hence, every IP starting with 86.x.x.x shall ignored.
10. Upon reviewing the auth log, found several IPs attempt to establish a connection to port 22 but is dropped by the system. It failed to connect.

> ONE OF THEM --> At `12:14:50 - 12:14:51`

![image](https://github.com/jon-brandy/hackthebox/assets/70703371/b36b6f92-d52c-49c9-abe5-d7e3b2fb994f)

#### NOTES:

```
The failed pre-auth behavior is often indicative of automated scanning or probing activity. It's a good practice to monitor
such activities and take preventive measures if necessary.
```

11. Upon further check about the IP. We found that the IP is registered as "Bad Reputation IP" with abuse reports of 15,037 times.

![image](https://github.com/jon-brandy/hackthebox/assets/70703371/9b6007de-864c-4912-ac3c-cbfd785a7859)


12. But we can't jump to a conclusion yet that it's the one that compromised the Forela's organization. Because there are bunch other failed logon with Bad Reputation IP.

13. Long story short, one IP caught my interest --> `195.80.150.137`. It's stuffed few creds at `10:59:12 - 11:08:09`.

![image](https://github.com/jon-brandy/hackthebox/assets/70703371/ebfebec5-c4c9-4282-81c9-5856f6ca4ee7)


14. But then it authenticated as `grafana` at `11:17:18`. Nice, we hunted one IP then. The rest IP are still the suspect.

![image](https://github.com/jon-brandy/hackthebox/assets/70703371/88feca18-1fcf-40b5-8052-ca89e1dfacfc)

15. Now, let's review the `Process_and_Network` and let's query brutally for XMRIG process. We just want to get more information about the XMRIG activities at the EC2.

> RESULTS - Got few interesting informations.

![image](https://github.com/jon-brandy/hackthebox/assets/70703371/193efa2b-116a-402b-981f-69c4d5af8295)

16. Great! Based from the results above, we confirm that we can see a process named XMRIG running from `/usr/share/logs.txt/`.
17. Now let's move to the grafana directory and review the logs at `/usr/share/grafana/data/log/`.

![image](https://github.com/jon-brandy/hackthebox/assets/70703371/9c3860e5-21f9-43a9-b208-79b71f4298f0)


18. Upon reviewing each logs, we will find several False Positive activity. Remembering the Forela currently running VA and maintenance to the server.

> FALSE POSITIVE IP --> each IP started with 86.x.x.x or IP with the same beginning 3 blocks.

![image](https://github.com/jon-brandy/hackthebox/assets/70703371/d824a932-64d6-42a6-ad1a-612ac1654e18)


19. Long story short, found another IP that attempted arbitrary file read.

![image](https://github.com/jon-brandy/hackthebox/assets/70703371/160de78c-a1d0-4bb4-b363-b504e2e5060f)


> FURTHER CHECK FOR THE IP





> 3RD QUESTION --> ANS:

![image](https://github.com/jon-brandy/hackthebox/assets/70703371/fb666590-22e3-46ab-b8e1-4aeda573da07)


> 4TH QUESTION --> ANS:

![image](https://github.com/jon-brandy/hackthebox/assets/70703371/c06d69a4-fe98-4513-966d-b003e7414f39)


> 5TH QUESTION --> ANS:

![image](https://github.com/jon-brandy/hackthebox/assets/70703371/a4bca183-54a5-42ac-ae59-eaaae9304e56)


> 6TH QUESTION --> ANS:

![image](https://github.com/jon-brandy/hackthebox/assets/70703371/267a1ade-f604-466c-870c-8e15e48a63a2)


> 7TH QUESTION --> ANS:

![image](https://github.com/jon-brandy/hackthebox/assets/70703371/df8aea36-be13-497e-aa09-bb3acc8b61a7)


> 8TH QUESTION --> ANS:

![image](https://github.com/jon-brandy/hackthebox/assets/70703371/28f31da6-a23f-42e6-8d00-c177374db3bd)


> 9TH QUESTION --> ANS:

![image](https://github.com/jon-brandy/hackthebox/assets/70703371/d807380d-67a5-4327-9081-020ca2b05c40)


> 10TH QUESTION --> ANS:

![image](https://github.com/jon-brandy/hackthebox/assets/70703371/cd6e1342-9d44-44aa-96d8-652531693d1f)


> 11TH QUESTION --> ANS:

![image](https://github.com/jon-brandy/hackthebox/assets/70703371/250bdae2-45b4-4e82-a44b-208640d673fa)


> 12TH QUESTION --> ANS:

![image](https://github.com/jon-brandy/hackthebox/assets/70703371/58dbbdaf-3c95-482b-9d90-f1036a6dee36)


> 13TH QUESTION --> ANS:

![image](https://github.com/jon-brandy/hackthebox/assets/70703371/ab23efbd-6abb-4ac0-8d25-3ff2a92a22c0)


> 14TH QUESTION --> ANS:

![image](https://github.com/jon-brandy/hackthebox/assets/70703371/1c04868c-0311-48e1-9231-af5db3b86fea)


## IMPORTANT LINKS:

```
https://www.cybersecurity-help.cz/vdb/grafana_labs/grafana/8.2.0/
https://github.com/pedrohavay/exploit-grafana-CVE-2021-43798
```