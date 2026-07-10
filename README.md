# Automated Node.js Deployment Pipeline with GitHub Actions and Ansible

Bu proje; bir Node.js (Express) servisinin bulut altyapısının yönetimini, sunucu konfigürasyonunu ve sürekli dağıtım (CD) süreçlerini tam otomatik hale getiren uçtan uca bir DevOps hattıdır. 

Projede GitOps prensipleri benimsenmiş olup, ana repoya yapılan her güvenli gönderimde (push) GitHub Actions ve Ansible ortaklığıyla sıfır kesinti (zero-downtime) yaklaşımına uygun bir dağıtım gerçekleştirilir.

---

## Sistem Mimarisi ve Teknoloji Yığınları

- **Uygulama Katmanı:** Minimalist Express.js mimarisiyle yapılandırılmış, 3000 portunda dinleme yapan Node.js servisi.
- **Altyapı (Infrastructure):** Terraform IaC scriptleri aracılığıyla DigitalOcean üzerinde dinamik olarak ayağa kaldırılan Ubuntu 24.04 LTS Droplet.
- **Konfigürasyon Yönetimi (Ansible):** Sunucu içi paketlerin (Node.js v22, NPM, PM2, Nginx) kurulması ve servis ayarlarının yapılması.
- **Ters Vekil Sunucu (Nginx Reverse Proxy):** 80 portundan gelen harici HTTP isteklerini karşılayarak içeride çalışan 3000 portundaki Node.js servisine güvenli bir şekilde yönlendirir.
- **Proses Yönetimi (PM2):** Node.js uygulamasının arka planda (daemon) sürekli çalışmasını, çökme durumlarında otomatik yeniden başlamasını ve deployment esnasında eski süreçlerin temizlenmesini yönetir.
- **CI/CD Orkestrasyonu (GitHub Actions):** Kod değişikliklerini algılayıp Ansible playbook'unu tetikleyen otomasyon merkezi.

---

## CI/CD İş Akışı (Workflow)



1. Geliştirici yerel makinesinde kodu günceller ve `main` branch'ine pushlar.
2. GitHub Actions üzerinde tanımlı iş akışı (workflow) tetiklenir ve sanal bir Ubuntu ortamı ayağa kalkar.
3. GitHub Actions, şifreli Secrets katmanından aldığı `SSH_PRIVATE_KEY` ile kimlik doğrulaması gerçekleştirir.
4. Merkezi konfigürasyon reposu olan `configuration-management` SSH protokolü üzerinden Actions ortamına klonlanır.
5. Dinamik bir `inventory.ini` dosyası oluşturularak hedef sunucunun IP adresi Ansible'a tanıtılır.
6. `ansible-playbook setup.yml --tags "app"` komutu koşturularak sunucu üzerindeki ilgili rol tetiklenir.
7. Ansible sunucuya bağlanır, güncel kodları çeker, PM2 üzerindeki eski süreci hafızadan kazır (`pm2 delete`) ve yeni kodu sıfırdan canlıya alır.

---

## GitHub Secrets Yapılandırması

Pipeline'ın kararlı çalışabilmesi için hedef depoda (Repository Secrets) aşağıdaki değişkenlerin tanımlanması zorunludur:

- `SERVER_IP`: Terraform tarafından üretilen ve Nginx'in dinlediği halka açık bulut sunucu IP adresi.
- `SSH_PRIVATE_KEY`: Sunucuya ve ilişkili private GitHub repolarına erişim yetkisine sahip olan OpenSSH formatındaki özel anahtar.

---

## DevOps Kazanımları ve Standartlar

- **Secret Management:** API token'ları ve SSH anahtarları kod içerisine gömülmemiş (hardcoded), ortam değişkenleri ve GitHub Secrets altyapısı kullanılarak tamamen izole edilmiştir.
- **Idempotency (Eşgüçlülük):** Playbook'lar kaç kez tetiklenirse tetiklensin, sunucu her zaman hedef durumunu korur ve gereksiz kaynak tüketiminin önüne geçilir.
- **Separation of Concerns (Sorumlulukların Ayrılması):** Altyapıyı kuran katman (Terraform), sunucuyu yapılandıran katman (Ansible) ve uygulama katmanı (Node.js) tamamen bağımsız modüller olarak tasarlanmıştır.

---

## Geliştirici

- **Aziz Mevlana Alim** - [GitHub](https://github.com/aziz-mevlana) - [Roadmap](https://roadmap.sh/projects/nodejs-service-deployment)
