# نشر وصّل (للمطوّر أو للمرة الأولى)

المتطلبات: خادم VPS (Linux) فيه Docker و Docker Compose، ودومين اختياري. لا تستخدم استضافة بلا قرص دائم (مثل Vercel) لأن الصور تُحفظ محليًا.

1. ارفع مجلد المشروع إلى الخادم.
2. أنشئ ملف `.env` بجانب docker-compose.yml:
   ```
   DB_PASSWORD=<كلمة مرور قوية لقاعدة البيانات>
   APP_SECRET=<32 بايت base64: node -e "console.log(require('crypto').randomBytes(32).toString('base64'))">
   ```
3. شغّل: `docker compose up -d --build`
4. أنشئ حساب المالك (مرة واحدة):
   `docker compose run --rm -e OWNER_PHONE=+222XXXXXXXX -e OWNER_NAME="الاسم" -e OWNER_PASSWORD="كلمة مرور 12+ حرفًا" app npm run db:bootstrap`
5. افتح الموقع، ادخل بحساب المالك، غيّر كلمة المرور، فعّل المصادقة الثنائية من /security.
6. من الإعدادات: أضف المدن والمناطق ورسوم التوصيل ورقم واتساب، ثم الأقسام والمنتجات.

ضع أمام التطبيق وكيل HTTPS (Caddy أو Nginx) قبل فتحه للزبائن. خذ نسخة احتياطية دورية من volume قاعدة البيانات ومن volume الصور.
لا تستخدم `npm run db:seed` في الإنتاج: يُنشئ حسابات تجريبية بكلمات مرور معروفة.
