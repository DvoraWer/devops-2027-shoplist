# שיעור 3 — תרגילים
## Docker Compose, Nginx ו-volumes

**הנחיות:**
- לפתור לפי הסדר
- להסתכל במדריך רק אם נתקעתן
- לבדוק כל תרגיל לפני שעוברים לבא
- הפקודות רצות בטרמינל שבחרתן, **מתוך התיקייה `docker`**, אלא אם כתוב אחרת

---

## תרגיל 1 — מה קורה ל-frontend לבד? (קל)

**המשימה:**
1. מהתיקייה הראשית של הריפו, להריץ את ה-frontend בלי Compose:
   `docker run -d -p 8081:80 --name front-alone shoplist-frontend:local`
2. להריץ `docker ps -a --filter name=front-alone`. מה הסטטוס?
3. לקרוא את הלוג: `docker logs front-alone`
4. לנקות: `docker rm -f front-alone`

**מה רואים כאן:**
ה-container נפל מיד. בלוג כתוב `host not found in upstream "stats"`. Nginx מחפש את השמות `stats` ו-`backend` כבר בהפעלה, והשמות האלה קיימים רק בתוך הרשת של Compose. אותו image, בתוך Compose, עובד מצוין.

**שאלה:** למה ה-502 של stats בשיעור 2 וה-crash הזה הם בעצם אותה בעיה?

---

## תרגיל 2 — להציץ למסד הנתונים (בינוני)

**המשימה:**
1. להרים את המערכת: `docker compose up -d`
2. להוסיף בדפדפן שלושה מוצרים.
3. להיכנס ל-postgres ולשאול אותו ישירות:
   ```shell
   docker compose exec postgres psql -U shopuser -d shoplist -c "SELECT * FROM products;"
   ```
4. לסמן מוצר אחד כנקנה, ולמחוק מוצר אחר. להריץ שוב את הפקודה.

**מה רואים כאן:**
`docker compose exec` מריץ פקודה בתוך service שרץ, כמו `docker exec` משיעור 2, רק לפי שם ה-service. `psql` הוא הכלי של PostgreSQL, והוא כבר בתוך ה-image. הנתונים שהוספתן בדפדפן עברו דרך Nginx, ה-backend, ועד הטבלה.

> **ב-Git Bash:** אם מופיע `the input device is not a TTY`, להוסיף `winpty` בהתחלה, או להוסיף `-T` אחרי `exec`.

**בדיקה:** שלוש שורות בטבלה. אחרי השינויים: שתי שורות, ובאחת מהן `bought` הוא `t` (true).

---

## תרגיל 3 — בלי healthcheck (בינוני)

**המשימה:**
1. `docker compose down -v`, כדי להתחיל ממסד נתונים ריק.
2. ב-`docker-compose.yml`, למחוק את כל הבלוק `healthcheck:` של postgres, ולשנות את `condition: service_healthy` ל-`condition: service_started`.
3. `docker compose up -d`, ומיד אחרי זה: `docker compose logs backend`
4. לפתוח את `http://localhost:8080` מהר, תוך שניות מה-`up`.
5. להחזיר את הקובץ לגרסה המקורית: `git checkout docker-compose.yml` (עובד אם עשיתן commit בשלב 8 של המדריך. אחרת, להחזיר את השורות ידנית)

**מה רואים כאן:**
ה-backend עולה לפני ש-postgres מוכן. בקשות בשניות הראשונות נכשלות עם שגיאת חיבור בלוג (`connection refused` או `the database system is starting up`). עם ה-healthcheck, ה-backend מחכה, והבעיה לא קורה.

> לפעמים המחשב מהיר מספיק ולא רואים שגיאה. זה בדיוק הסיכון: זה עובד אצלי, ונכשל בשרת.

---

## תרגיל 4 — סיסמאות מחוץ לקובץ (בינוני)

**הבעיה:** הסיסמה `shoppass` כתובה ב-`docker-compose.yml`, והקובץ נכנס ל-Git. זה בדיוק מה שלמדנו לא לעשות בשיעור 2.

**המשימה:**
1. ליצור בתיקייה `docker` קובץ בשם `.env`:
   ```
   POSTGRES_PASSWORD=shoppass
   ```
2. ב-`docker-compose.yml`, להחליף את **שני** המקומות שבהם כתוב `shoppass` ב-`${POSTGRES_PASSWORD}`:
   ```yaml
   POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
   ```
   ```yaml
   DB_PASSWORD: ${POSTGRES_PASSWORD}
   ```
3. לבדוק מה Compose רואה: `docker compose config`. הסיסמה מופיעה במקום המשתנה.
4. `docker compose up -d` ולבדוק שהכל עובד.
5. לבדוק ש-Git לא רואה את `.env`: `git status`. אם הוא מופיע, להוסיף שורה `.env` לקובץ `.gitignore` בתיקייה הראשית.

**מה רואים כאן:**
Compose קורא אוטומטית קובץ בשם `.env` מאותה תיקייה, ומחליף כל `${...}` בערך שלו. הקובץ עם הסיסמה נשאר במחשב, והקובץ שנכנס ל-Git מכיל רק את שם המשתנה.

> **ב-Notepad:** לשמור עם "All files", בשם `.env` בדיוק. Notepad נוטה לשמור `.env.txt`.

---

## תרגיל 5 — multi-stage: לבנות בשלב אחד, להריץ בשלב אחר (בינוני)

**הרקע:**
באתרים אמיתיים הקבצים שהדפדפן מקבל לא נכתבים ביד. כלי בנייה (למשל Vite או webpack) מכין אותם, והכלי הזה רץ על Node.js. אבל כדי **להגיש** את הקבצים צריך רק Nginx. אין סיבה ש-Node, וכל כלי הבנייה, יישבו ב-image שרץ בשרת.

ב-`app/frontend` יש קובץ חדש, `build.js`. זה כלי בנייה קטן: הוא מעתיק את הקבצים לתיקייה `dist`, מוחק הערות מ-`app.js`, ומוסיף לתחתית הדף את זמן הבנייה. אין לכן Node במחשב, וזה לא משנה: הבנייה תרוץ בתוך container.

**המשימה:**
1. ליצור בתיקייה `app/frontend` קובץ בשם `Dockerfile.multistage`:
   ```dockerfile
   # שלב 1: בנייה, עם Node
   FROM node:24-alpine AS build
   WORKDIR /app
   COPY build.js index.html style.css app.js ./
   RUN node build.js

   # שלב 2: הרצה, עם Nginx בלבד
   FROM nginx:1.30-alpine
   COPY --from=build /app/dist/ /usr/share/nginx/html/
   COPY nginx.conf /etc/nginx/conf.d/default.conf
   EXPOSE 80
   ```
2. לבנות, **מהתיקייה הראשית של הריפו**. הדגל `-f` אומר באיזה Dockerfile להשתמש:
   ```shell
   docker build -f app/frontend/Dockerfile.multistage -t shoplist-frontend:multi app/frontend
   ```
   בפלט של ה-build רואים את שני השלבים, ואת השורה `node build.js` עם רשימת הקבצים.
3. לבנות רק את השלב הראשון, עם `--target`:
   ```shell
   docker build -f app/frontend/Dockerfile.multistage --target build -t shoplist-frontend:build-stage app/frontend
   ```
4. להשוות גדלים, ולרשום אותם:
   ```shell
   docker images shoplist-frontend
   ```
5. לבדוק שאין Node ב-image הסופי:
   ```shell
   docker run --rm shoplist-frontend:build-stage node --version
   docker run --rm shoplist-frontend:multi node --version
   ```
   הראשון מדפיס גרסה. השני נכשל עם הודעה ש-`node` לא נמצא.
6. להשתמש ב-Dockerfile החדש ב-Compose. ב-`docker/docker-compose.yml`, ב-service `frontend`:
   ```yaml
   frontend:
     build:
       context: ../app/frontend
       dockerfile: Dockerfile.multistage
   ```
   ואז, מהתיקייה `docker`:
   ```shell
   docker compose up -d --build frontend
   ```
   לרענן את הדפדפן עם Ctrl+F5. בתחתית הדף מופיעה שורת `Build:` עם התאריך והשעה.

**מה רואים כאן:**
ב-Dockerfile יש שני `FROM`, וכל אחד מתחיל שלב חדש מאפס. `AS build` נותן שם לשלב הראשון, ו-`COPY --from=build` לוקח ממנו **רק** את התיקייה `dist`. כל השאר, כולל Node עצמו, נשאר מאחור ולא נכנס ל-image הסופי. לכן הוא קטן יותר, מהיר יותר להורדה, ויש בו פחות תוכנה שאפשר לתקוף.

**שאלות:**
1. כמה MB חסכתן? (מתוך הפלט של שלב 4)
2. למה `node --version` נכשל ב-image הסופי, אם `node build.js` רץ בזמן ה-build?
3. ב-Dockerfile הרגיל של ה-frontend אין שלב בנייה. מתי כדאי לעבור ל-multi-stage?

> **קיבלתן "Welcome to nginx!"?** השורה `COPY nginx.conf` חסרה בשלב השני. כל שלב מתחיל מאפס, גם ההגדרות.

---

## תרגיל בונוס — מוצר אחד לפי מספר (רשות)

**המשימה:**
להוסיף ל-`app/backend/app.py` כתובת חדשה:

```
GET /products/<id>   →  מחזיר מוצר אחד כ-JSON
                     →  מחזיר 404 עם {"error": "not found"} אם אין כזה
```

**רמז:** להעתיק את `get_products` ולשנות אותה. בקוד כבר יש `COLUMNS` (רשימת העמודות) ו-`to_json` (הופכת שורה ל-JSON). השאילתה: `SELECT ... FROM products WHERE id = %s`, ו-`cur.fetchone()` מחזיר שורה אחת, או `None` אם אין.

ואז, מהתיקייה `docker`:

```shell
docker compose up -d --build backend
curl.exe -i http://localhost:8080/api/products/1
curl.exe -i http://localhost:8080/api/products/9999
```

**מה רואים כאן:** `--build backend` בונה מחדש רק את ה-backend. בזכות ה-cache משיעור 2, זה לוקח שניות. שאר הסרוויסים לא נגעו.

> **קיבלתן `502 Bad Gateway`?** ה-backend נוצר מחדש, ולפעמים הוא מקבל כתובת חדשה ברשת. Nginx זוכר את הכתובת הישנה מהרגע שהוא עלה. הפתרון: `docker compose restart frontend`, ואז שוב `curl`.
