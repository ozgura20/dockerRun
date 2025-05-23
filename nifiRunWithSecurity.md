Aşağıda sana, bu süreci baştan sona belgeleyen ve Git'e koyabileceğin şekilde hazırlanmış bir README.md dosyası şablonu veriyorum. Bu dosya:

Sertifika üretimi (PKCS12 ve SAN içeren),

Docker ile NiFi’nin güvenli (HTTPS) kurulumu,

Windows hosts ayarları,

Ve hatalardan kaçınma notlarını içeriyor.

📄 README.md
markdown
Copy
Edit
# Apache NiFi HTTPS Docker Kurulumu (PKCS12 + SAN Sertifikası)

## 🧩 Gereksinimler

- Docker yüklü bir Linux makinesi
- Windows istemci
- OpenSSL ve `keytool` kurulu
- NiFi `latest` Docker imajı

---

## 📍 1. Sertifika Oluştur (PKCS12 + SAN)

```bash
mkdir -p ~/nifi-certs

# JKS sertifikayı oluştur
keytool -genkeypair -alias nifi-key -keyalg RSA -keysize 2048 \
  -storetype JKS -keystore ~/nifi-certs/keystore.jks \
  -storepass admin123 -keypass admin123 \
  -dname "CN=nifi.local, OU=NiFi, O=Apache, L=Local, S=State, C=TR" \
  -ext SAN=dns:nifi.local,ip:192.168.147.116

# PKCS12 formatına çevir
keytool -importkeystore \
  -srckeystore ~/nifi-certs/keystore.jks \
  -destkeystore ~/nifi-certs/keystore.p12 \
  -srcstoretype JKS -deststoretype PKCS12 \
  -srcstorepass admin123 -deststorepass admin123
🐳 2. Docker ile NiFi HTTPS Başlat
bash
Copy
Edit
docker run --name nifi \
  -v ~/nifi-certs:/opt/certs \
  -p 8443:8443 \
  -e NIFI_WEB_HTTPS_HOST=0.0.0.0 \
  -e NIFI_WEB_HTTPS_PORT=8443 \
  -e KEYSTORE_PATH=/opt/certs/keystore.p12 \
  -e KEYSTORE_TYPE=PKCS12 \
  -e KEYSTORE_PASSWORD=admin123 \
  -e TRUSTSTORE_PATH=/opt/certs/keystore.p12 \
  -e TRUSTSTORE_TYPE=PKCS12 \
  -e TRUSTSTORE_PASSWORD=admin123 \
  -e NIFI_WEB_PROXY_HOST="nifi.local,192.168.147.116" \
  -e SINGLE_USER_CREDENTIALS_USERNAME=admin \
  -e SINGLE_USER_CREDENTIALS_PASSWORD=SüperGüçlüParola123 \
  -d apache/nifi:latest
🖥️ 3. Windows hosts Ayarı
makefile
Copy
Edit
C:\Windows\System32\drivers\etc\hosts
Şu satırı en alta ekle:

lua
Copy
Edit
192.168.147.116   nifi.local
✅ 4. Erişim
Tarayıcıdan şuraya git:

bash
Copy
Edit
https://nifi.local:8443/nifi/
⚠️ Sertifika self-signed olduğu için ilk girişte "Güvenli değil" uyarısı verebilir. Devam etmeniz yeterli.

🛠️ Sorun Giderme
HTTP ERROR 400 Invalid SNI
Sertifikadaki CN veya SAN alanı nifi.local değilse bu hata alınır.

NIFI_WEB_PROXY_HOST=nifi.local ayarı eksikse bu hataya neden olur.

curl ile test etmek için:
bash
Copy
Edit
curl -vk --resolve nifi.local:8443:192.168.147.116 https://nifi.local:8443/nifi/
🔐 Güvenlik Notları
admin123 gibi şifreleri sadece test ortamında kullanın.

Gerçek ortamda geçerli CA ile imzalanmış sertifikaları tercih edin.

✍️ Hazırlayan
Özgür – [ChatGPT ile desteklenmiştir]

yaml
Copy
Edit

---

İstersen bu içeriği `.md` dosyası olarak doğrudan verebilirim.  
Hazırsa Git'e atılacak haliyle `.zip` yapabiliriz veya Markdown dosyasını oluşturabilirim. Hangisini istersin?










Tools



ChatGPT can make mistakes. Check important info.