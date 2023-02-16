## **TryHackMe : ATTACKTIVE DIRECTORY YOL HARITASI** 
![](https://tryhackme.com/room/attacktivedirectory)
THM'den (TryHackMe'den) oğrendiğim bilgileri unutmamak için bir yol haritası oluşturuyorum. Bu yol haritasını izlemeden önce kerbrute programını bilgisayarınıza indiriniz. [Buradan wget kullanarak bilgisayarınıza indirebilirsiniz](https://github.com/ropnop/kerbrute/releases/download/v1.0.3/kerbrute_linux_amd64) Vakit kaybetmeden başlayalım:

### *Adım 1:*
İlk olarak kali'mizi acalım ve THM'den daha once indirmis oldugumuz vpn'i asagidaki komutla çalistiralim:
`openvpn <vpnismi>.ovpn`
THM'nin makinesini başlatalım.
### *Adım2:*
İmpacket kurulumu için aşağıdaki komutları sırasıyla terminalde çalıştıralım:

`git clone https://github.com/SecureAuthCorp/impacket.git`

`/opt/impacket`

`pip3 install -r /opt/impacket/requirements.txt`

`cd /opt/impacket/ && python3 ./setup.py install`
### *Adım3:*
Temel port taraması ve numaralandırma işlemi için nmap kullanarak başlayacağız. Sonrasında farklı numaralandırma işlemleri için farklı programları kullanacağız. Tüm bu işlemleri yapmadan önce elde ettiğimiz bilgileri kaydetmek için not.txt dosyasını açalım ve nmap taramasını yapalım:

`nmap -v -sS -A -O -T4 <Hedef IP>`

![](https://github.com/hamza37yavuz/AttacktiveD-rectory-YolHaritas-/blob/main/nmap.png)

THM'deki soruların cevaplarını not.txt dosyasını inceleyerek bulabilirsiniz.

139 ve 445 numaralı bağlantı noktaları SMB tarafından kullanılır. SMB'yi numaralandırmak için enum4linux'u kullanacağız.

`enum4linux <hedef ip> -a spookysec.local`

![](https://github.com/hamza37yavuz/AttacktiveD-rectory-YolHaritas-/blob/main/enum4linux.png)

Yukarıdaki resimde kullanıcı grupları listelenmiştir yine bu kullanıcı grupları not.txt'dosyasına kaydedilmiştir.

### *Adım 5:*

Kerberos da dahil olmak üzere bir dizi başka hizmet çalışıyor . Kerberos, Active Directory içindeki bir anahtar kimlik doğrulama hizmetidir. Bu bağlantı noktası açıkken, kullanıcıların parolalarını ve hatta parola spreyini kaba kuvvetle keşfi mümkündür. Bunun için Kerbrute (Ronnie Flathers @ropnop tarafından oluşturulan) adlı bir araç kullanabiliriz !

Bu sistem için hazırlanmış kullanıcı listesi ve parola listesi, kullanıcıların numaralandırılması ve parola kırma süresini kısaltmak için kullanılacaktır. Bu listeleri `wget` kullanarak bilgisayarımıza indirebiliriz.

>[THM'nin kullanıcı listesi](https://raw.githubusercontent.com/Sq00ky/attacktive-directory-tools/master/userlist.txt)

>[THM'nin şifre listesi](https://raw.githubusercontent.com/Sq00ky/attacktive-directory-tools/master/passwordlist.txt)

Kerbrute kullanarak kullanıcı adlarını almak için `userenum` komutunu kullanacağız. Kullanımın nasıl olduğu farklı şekillerde nasıl kullanılabileceğini anlamak için doğru dosya dizinine giderek `./kerbrute -h` komutunu çalıştırabilirsiniz. Kullanıcıları görmek için aşağıdaki komutu terminalimizde çalıştıracağız.

`./kerbrute userenum userlist.txt --dc <hedef ip> -d spookysec.local`

![alt text](https://github.com/hamza37yavuz/Attacktive-Directory-YolHaritasi-/blob/main/kerbrute.png)
### *Adım 5:*
Kullanıcı hesaplarının numaralandırılması tamamlandıktan sonra, ASREPRoasting adlı bir saldırı yöntemi ile Kerberos içindeki bir özelliği kötüye kullanmaya çalışabiliriz. ASReproasting saldırısı bir kullanıcı hesabı "Ön Kimlik Doğrulaması Gerektirmez" ayrıcalığına sahip olduğunda gerçekleşir. Bu, hesabın belirtilen kullanıcı hesabından bir kerberos bileti talep etmeden önce geçerli bir kimlik sağlaması gerekmediği anlamına gelir.
Impacket , ASReproastable hesaplarını Anahtar Dağıtım Merkezinden sorgulamamıza izin verecek “GetNPUsers.py” adlı bir araca sahiptir (impacket/examples/GetNPUsers.py konumunda bulunur). Hesapları sorgulamak için gerekli olan tek şey, daha önce Kerbrute aracılığıyla sıraladığımız hesaplar arasından bir hesap seçmek ve bilet almak için deneme yapmaktır. 4. Adımda elde ettiğimiz listedeki kullanıcı adlarını burada kullanacağız.
Bunun için ilk olarak `/opt/impacket/examples` dizinine gitmeliyiz. Bu işlemi yaptıktan sonra getNPUsers.py dosyasını aşağıdaki gibi çalıştırarak şifre hash'ini (bileti) almaya çalışacağız.

`python GetNPUsers.py -dc-ip <hedef ip> spookysec.local/<kullanıcı adı> -no-pass` --> kullanıcı adı yerine svc-admin yazılabilir

hash'i elde ettik ve not.txt dosyasına kaydettik. Bu hash svc-admin kullanıcısının şifresinin hash'idir. Bu hash'i çözmek için hashcat programını kullanacağız. Hashcat programını çalıştırmak için mod değerine ihtiyacımız var bu mod değerini elde etmek için [bu siteye bakabilirsiniz.](https://hashcat.net/wiki/doku.php?id=example_hashes)
hashcat komutunu çalıştırmak için elde ettiğimiz hash'i bir txt'ye yapıştırmalıyız ve aşağıdaki şekilde yazarak şifreyi elde edebiliriz.

`hashcat -m 18200 hashCode.txt passwordlist.txt`
