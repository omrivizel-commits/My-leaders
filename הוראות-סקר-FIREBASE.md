# חיבור הסקר ל‑Firebase (הפעלה אמיתית)

עד שלא תחבר Firebase, הסקר רץ ב**מצב הדגמה** (נתונים מקומיים בלבד על המכשיר שלך).
כדי שכל טלפון יוכל להצביע פעם אחת והתוצאות יתאחדו לתמונת מצב אחת — בצע את השלבים הבאים פעם אחת.

## שלב 1 — יצירת פרויקט
1. כנס ל‑https://console.firebase.google.com והתחבר עם החשבון `omrivizel@gmail.com`.
2. לחץ **Add project** → תן שם (למשל `my-leaders-poll`) → אפשר לכבות Analytics → **Create project**.

## שלב 2 — מסד נתונים (Firestore)
1. בתפריט: **Build → Firestore Database → Create database**.
2. בחר **Production mode** → בחר מיקום (למשל `eur3`) → **Enable**.
3. עבור ללשונית **Rules**, מחק הכול והדבק את הכללים הבאים, ואז **Publish**:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    function isAdmin() {
      return request.auth != null && request.auth.token.email == 'omrivizel@gmail.com';
    }
    match /poll/meta {
      allow read: if true;
      allow write: if isAdmin();
    }
    match /votes/{deviceId} {
      allow read: if true;
      allow create: if request.resource.data.party is string
                    && request.resource.data.party.size() < 60;
      allow update, delete: if false;
    }
  }
}
```

## שלב 3 — התחברות מנהל (Authentication)
1. **Build → Authentication → Get started**.
2. בלשונית **Sign-in method** → הפעל **Email/Password** → **Save**.
3. בלשונית **Users → Add user** → אימייל `omrivizel@gmail.com` + סיסמה שתבחר → **Add**.
   (זו הסיסמה שאיתה תתחבר לניהול הסקר באתר.)

## שלב 4 — העתקת ההגדרות לאתר
1. לחץ על גלגל השיניים (Project settings) → **General**.
2. גלול ל‑**Your apps** → לחץ על אייקון ה‑Web `</>` → תן כינוי → **Register app**.
3. יוצג אובייקט `firebaseConfig` עם 6 ערכים. העתק אותם.
4. ב‑`index.html`, מצא את הבלוק `const firebaseConfig = { ... }` (בתחתית הקובץ, בסקריפט של הסקר)
   והחלף את כל ערכי `PASTE_...` בערכים שלך.
5. שמור, ועשה commit + push ל‑GitHub.

## שלב 5 — הפעלה
1. כנס לאתר עם הכתובת: `https://omrivizel-commits.github.io/My-leaders/?admin`
2. לחץ על כפתור 🔐 (למעלה) → התחבר עם האימייל והסיסמה משלב 3.
3. תראה את **לוח הניהול** עם מספר ההצבעות וכפתור **🚀 פרסם לציבור**.
4. עד שתלחץ "פרסם" — אף אחד חוץ ממך לא רואה את הסקר.
5. כשתלחץ "פרסם" — הסקר נפתח לכולם, וכל טלפון יכול להצביע פעם אחת.

---

### עריכת מפלגות / שאלה
ב‑`index.html` בתוך סקריפט הסקר יש מערך `PARTIES` (שם + צבע לכל מפלגה) ומשתנה `QUESTION`.
ערוך אותם כרצונך.

### מגבלות (חשוב לדעת)
- "קול אחד לכל טלפון" נאכף לפי מזהה שנשמר בדפדפן. מי שמנקה את נתוני הדפדפן או גולש בגלישה פרטית
  יכול להצביע שוב. זה סטנדרטי לסקר פתוח שלא דורש הרשמה.
- זהו סקר לא‑מדעי (מי שנכנס ומצביע), לא מדגם מייצג — מתאים כ"תמונת מצב" קהילתית.
- השכבה החינמית של Firebase (50K קריאות/יום) יותר מספיקה לסקר כזה.
