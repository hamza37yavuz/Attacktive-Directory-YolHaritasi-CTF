## **TryHackMe : ATTACKTIVE DIRECTORY YOL HARITASI** 
![](https://tryhackme.com/room/attacktivedirectory)
THM'den ogrendigim bilgileri unutmamak icin bir yol haritasi olusturuyorum.
Vakit kaybetmeden baslayayım:

#### *Adim 1:*
Ilk olarak kali'mizi acalım ve THM'den daha once indirmis oldugumuz vpn'i asagidaki komutla çalistiralim:
`openvpn <vpnismi>.ovpn`
THM'nin makinesini başlatalım.
#### *Adim2:*
İmpacket kurulumu için aşağıdaki komutları sırasıyla terminalde çalıştıralım:
`git clone https://github.com/SecureAuthCorp/impacket.git`
`/opt/impacket`
`pip3 install -r /opt/impacket/requirements.txt`
`cd /opt/impacket/ && python3 ./setup.py install`
### *Adim3:*
Temel port taraması ve numaralandırma için nmap kullanarak başlayacağız. Sonrasında farklı numaralandırma işlemleri için farklı programları kullanacağız. Tüm bu işlemleri yapmadan önce elde ettiğimiz bilgileri kaydetmek için not.txt dosyasını açalım. Şimdi nmap taramasını yapalım:
`nmap -v -sS -A -O -T4 <Hedef IP>`
