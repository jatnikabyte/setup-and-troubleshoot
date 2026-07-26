### Deploy Key (Read-only)

Jika akses hanya diperlukan dari **satu komputer/server**, gunakan **Deploy Key**.

1. Di komputer client:

   ```bash
   ssh-keygen -t ed25519
   ```

2. Salin isi:

   ```bash
   cat ~/.ssh/id_ed25519.pub
   ```

3. Di GitHub:

   ```
   Repository
   → Settings
   → Deploy Keys
   → Add deploy key
   ```

4. **Jangan centang** "Allow write access".

Hasilnya:

* ✅ Clone
* ✅ Pull
* ❌ Push
* 
---
### JPOS Script Update

## Langkah 1. Membuat Script

Buat file baru:

```bash
nano ~/update-jpos.sh
```

Isi dengan script berikut:

```bash
#!/bin/bash

set -e

echo "=================================="
echo "Updating JPOS..."
echo "=================================="

cd ~/project/jpos

echo "📥 Pulling latest source..."
git pull origin main

echo "🗄️ Running database migration..."
php artisan migrate --force

echo "🏗️ Building frontend assets..."
npm run build

echo "🧹 Clearing Laravel cache..."
php artisan optimize:clear

echo "=================================="
echo "✅ Update completed successfully!"
echo "=================================="
```

## Langkah 2. Memberikan Hak Eksekusi

```bash
chmod +x ~/update-jpos.sh
```

---

## Langkah 3. Menjalankan Script

```bash
~/update-jpos.sh
```

atau

```bash
bash ~/update-jpos.sh
```
