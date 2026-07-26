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
