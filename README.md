# Fire-and-Water-Mobile-Duo

حاوية Android مستقلة تشغّل ملف SWF الأصلي للعبة **Fireboy and Watergirl 2: The Light Temple** عبر **Ruffle** داخل WebView،
مع أزرار تحكم باللمس ووضع لعب ثنائي عبر Wi-Fi المحلي.
(Standalone Android wrapper that runs the original SWF through Ruffle, with touch controls and LAN two-phone play.)

## حالة المشروع (بصراحة)
| الميزة | الحالة |
|---|---|
| فحص SWF وتحليل ABC | **تم** – انظر `docs/ANALYSIS.md` |
| مشروع Android Studio (Kotlin) + GitHub Actions | **مكتوب، لم يُبنَ ولم يُختبر** (بيئة الإعداد بلا Android SDK/Maven) |
| تشغيل SWF عبر Ruffle على الجهاز | **غير مُختبر** – خطر معروف: إعلانات CPMStar/Armor Games قد توقف شاشة التحميل |
| أزرار اللمس | مكتوبة، **غير مختبرة** على جهاز |
| Multiplayer عبر Wi-Fi | **منفّذ كوداً فقط – لم يُختبر بين هاتفين**، لا تعتمد على أنه يعمل قبل تجربته |

## ملف اللعبة وحقوق النشر
اللعبة محمية بحقوق النشر لأصحابها. لذلك **لا يتضمن المستودع ملف SWF** (وهو مُستثنى في `.gitignore`).
لا تنشره في مستودع عام. لديك طريقتان:
1. **داخل التطبيق:** اضغط *Choose game file (.swf)* واختر ملفك (يُنسخ إلى تخزين التطبيق، الأصل لا يتغير).
2. **مضمّن في APK (للاستعمال الشخصي):** `scripts/import_swf.sh /path/to/game.swf` ثم ابنِ محلياً (أو في مستودع **خاص**).

## البناء
المتطلبات: JDK 17، Android SDK (API 34)، Node/npm، Gradle 8.7+ (أو `gradle wrapper` لتوليد `gradlew`).
```bash
bash scripts/build_apk.sh      # ينتج dist/Fire-and-Water-Mobile-Duo-debug.apk
```
عبر GitHub: ادفع إلى `main` أو شغّل Workflow **Build APK** من تبويب Actions ثم نزّل الـ artifact.
APK الـ debug موقّع بمفتاح debug ويُثبَّت مباشرة (فعّل «التثبيت من مصادر غير معروفة»).

## التوافق
Android 14 (targetSdk 34, minSdk 24)، armeabi-v7a: لا يوجد كود native في التطبيق (Ruffle يعمل كـ WebAssembly داخل WebView النظام)،
لذا الـ APK لا يرتبط بمعمارية معينة. يتطلب WebView حديثاً يدعم WebAssembly. لا يحتاج Root.

## اللعب الثنائي (Wi-Fi)
تصميم: الهاتف **المضيف** يشغّل اللعبة ويتحكم بـ **Fireboy**؛ الهاتف **الضيف** يعمل كذراع تحكم لاسلكي لـ **Watergirl**
(لا بث للفيديو – الضيف لا يرى اللعبة). التزامن بسيط: الضيف يرسل أحداث الضغط/الإفلات فقط.
1. المضيف: *Host* ← يظهر PIN وIP والمنفذ ويعلن الخدمة عبر mDNS (`_fwduo._tcp`).
2. الضيف: *Join* ← أدخل PIN ← *Find host automatically*. إن حجب الراوتر mDNS استعمل IP/المنفذ يدوياً.
3. الشبكة نفسها مطلوبة، وعزل العملاء (AP isolation) يمنع الاتصال. لا تشفير؛ استعمل شبكة موثوقة. المضيف يقبل مفاتيح Watergirl فقط.

## الهيكل
```
app/src/main/java/com/fwduo/   MainActivity, LanHost, LanClient, LanProtocol
app/src/main/assets/web/       index.html (قائمة + تحكم لمس + Ruffle)  [ruffle/ تُجلب بالسكربت]
app/src/main/assets/game/      ضع game.swf هنا محلياً (مستثنى من git)
scripts/                       fetch_ruffle.sh, import_swf.sh, build_apk.sh
.github/workflows/build-apk.yml
docs/ANALYSIS.md
```

## التراخيص
* هذا الكود: اختر ترخيصك (لم يُضف ملف LICENSE بعد).
* Ruffle 0.7.1: MIT أو Apache-2.0 (تُنسخ ملفات LICENSE_* مع الأصول).
* AndroidX / Kotlin: Apache-2.0.
* اللعبة الأصلية: ملك أصحابها، غير مرخّصة هنا.

## إن لم تعمل اللعبة
افتح `chrome://inspect` على الحاسوب مع الهاتف (USB debugging) لرؤية console الـ WebView. إن توقفت عند التحميل بسبب الإعلانات
فالحل البديل: تعديل نسخة من SWF (ليس الأصل) لتجاوز مُحمّل الإعلانات، أو إعادة كتابة اللعبة – وهذا لم يُنفّذ.
