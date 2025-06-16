# GitHub Copilot Metrics Dashboard

[English](#english) | [Türkçe](#turkish)

## English

### Overview
GitHub Copilot Metrics Dashboard is a web application that provides insights and analytics for GitHub Copilot usage. It helps teams track and analyze their Copilot usage patterns, efficiency, and impact on development workflows.

### Features
- Real-time metrics visualization
- Usage statistics and trends
- Team performance analytics
- Customizable dashboards
- Secure authentication with GitHub OAuth

### Prerequisites
- Node.js (v16 or higher)
- npm or yarn
- GitHub account with Copilot access
- Docker and Docker Compose (for containerized deployment)

### Installation
1. Clone the repository:
```bash
git clone https://github.com/yourusername/github-copilot-metrics-dashboard.git
cd github-copilot-metrics-dashboard
```

2. Install dependencies:
```bash
npm install
```

3. Configure environment variables:
```env
NUXT_GITHUB_TOKEN=your_github_token
NUXT_SESSION_PASSWORD=your_session_password
NUXT_OAUTH_GITHUB_CLIENT_SECRET=your_github_client_secret
```

4. Start the development server:
```bash
npm run dev
```

### Docker Support
#### Using Docker
Build and run with Docker:
```bash
docker build -t copilot-metrics-dashboard .
docker run -p 3000:3000 copilot-metrics-dashboard
```

#### Using Docker Compose
1. Create a `.env` file with your environment variables:
```env
NUXT_GITHUB_TOKEN=your_github_token
NUXT_SESSION_PASSWORD=your_session_password
NUXT_OAUTH_GITHUB_CLIENT_SECRET=your_github_client_secret
```

2. Start the application:
```bash
docker-compose up -d
```

3. To stop the application:
```bash
docker-compose down
```

### Contributing
Contributions are welcome! Please feel free to submit a Pull Request.

### License
MIT License

---

## Turkish

### Genel Bakış
GitHub Copilot Metrics Dashboard, GitHub Copilot kullanımı için içgörüler ve analitikler sağlayan bir web uygulamasıdır. Ekiplerin Copilot kullanım modellerini, verimliliğini ve geliştirme iş akışlarına etkisini takip etmelerine ve analiz etmelerine yardımcı olur.

### Özellikler
- Gerçek zamanlı metrik görselleştirme
- Kullanım istatistikleri ve trendler
- Ekip performans analitiği
- Özelleştirilebilir paneller
- GitHub OAuth ile güvenli kimlik doğrulama

### Gereksinimler
- Node.js (v16 veya üzeri)
- npm veya yarn
- Copilot erişimi olan GitHub hesabı
- Docker ve Docker Compose (konteynerli dağıtım için)

### Kurulum
1. Depoyu klonlayın:
```bash
git clone https://github.com/yourusername/github-copilot-metrics-dashboard.git
cd github-copilot-metrics-dashboard
```

2. Bağımlılıkları yükleyin:
```bash
npm install
```

3. Ortam değişkenlerini yapılandırın:
```env
NUXT_GITHUB_TOKEN=github_tokeniniz
NUXT_SESSION_PASSWORD=oturum_sifreniz
NUXT_OAUTH_GITHUB_CLIENT_SECRET=github_client_secretiniz
```

4. Geliştirme sunucusunu başlatın:
```bash
npm run dev
```

### Docker Desteği
#### Docker Kullanımı
Docker ile derleyip çalıştırın:
```bash
docker build -t copilot-metrics-dashboard .
docker run -p 3000:3000 copilot-metrics-dashboard
```

#### Docker Compose Kullanımı
1. `.env` dosyası oluşturun ve ortam değişkenlerinizi ekleyin:
```env
NUXT_GITHUB_TOKEN=github_tokeniniz
NUXT_SESSION_PASSWORD=oturum_sifreniz
NUXT_OAUTH_GITHUB_CLIENT_SECRET=github_client_secretiniz
```

2. Uygulamayı başlatın:
```bash
docker-compose up -d
```

3. Uygulamayı durdurmak için:
```bash
docker-compose down
```

### Katkıda Bulunma
Katkılarınızı bekliyoruz! Lütfen Pull Request göndermekten çekinmeyin.

### Lisans
MIT Lisansı

## Örnek Dashboard

### Türkçe
Aşağıda örnek dashboard görüntüsü yer almaktadır:

![Örnek Dashboard](./public/example-dashboard.png)

### English
Below is an example dashboard image:

![Example Dashboard](./public/example-dashboard.png)
