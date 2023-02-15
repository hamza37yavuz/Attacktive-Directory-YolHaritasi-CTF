## **TryHackMe : ATTACKTIVE DIRECTORY YOL HARITASI** 
![](https://tryhackme.com/room/attacktivedirectory)
THM'den ogrendigim bilgileri unutmamak icin bir yol haritasi olusturuyorum.
Vakit kaybetmeden baslayayım:

### *Adım 1:*
Ilk olarak kali'mizi acalım ve THM'den daha once indirmis oldugumuz vpn'i asagidaki komutla çalistiralim:
`openvpn <vpnismi>.ovpn`
THM'nin makinesini başlatalım.
### *Adım2:*
İmpacket kurulumu için aşağıdaki komutları sırasıyla terminalde çalıştıralım:

`git clone https://github.com/SecureAuthCorp/impacket.git`

`/opt/impacket`

`pip3 install -r /opt/impacket/requirements.txt`

`cd /opt/impacket/ && python3 ./setup.py install`
### *Adım3:*
Temel port taraması ve numaralandırma için nmap kullanarak başlayacağız. Sonrasında farklı numaralandırma işlemleri için farklı programları kullanacağız. Tüm bu işlemleri yapmadan önce elde ettiğimiz bilgileri kaydetmek için not.txt dosyasını açalım. Şimdi nmap taramasını yapalım:
`nmap -v -sS -A -O -T4 <Hedef IP>`

THM'deki soruların cevaplarını not.txt dosyasını inceleyerek bulabilirsiniz.

139 ve 445 numaralı bağlantı noktaları SMB tarafından kullanılır. SMB'yi numaralandırmak için enum4linux'u kullanacağız.

`enum4linux <hedef ip> -a spookysec.local`

### *Adım4:*

Kerberos da dahil olmak üzere bir dizi başka hizmet çalışıyor . Kerberos, Active Directory içindeki bir anahtar kimlik doğrulama hizmetidir. Bu bağlantı noktası açıkken, kullanıcıların parolalarını ve hatta parola spreyini kaba kuvvetle keşfi mümkündür. Bunun için Kerbrute (Ronnie Flathers @ropnop tarafından oluşturulan) adlı bir araç kullanabiliriz !

Bu sistem için hazırlanmış kullanıcı listesi ve parola listesi, kullanıcıların numaralandırılması ve parola kırma süresini kısaltmak için kullanılacaktır. Bu listeleri `wget` kullanarak bilgisayarımıza indirebiliriz.

>![THM'nin önerdiği kullanıcı listesi](https://raw.githubusercontent.com/Sq00ky/attacktive-directory-tools/master/userlist.txt)

>![THM'nin önerdiği şifre listesi](https://raw.githubusercontent.com/Sq00ky/attacktive-directory-tools/master/passwordlist.txt)

