# שיעור 4 — שיעורי בית

**מה צריך:** דפדפן וחשבון GitHub. **לא צריך להתקין כלום בבית.** כותבות באתר של GitHub, ובודקות בכיתה בתחילת שיעור 5.

**זמן משוער:** שעה וחצי.

**מה מגישים ב-fork שלכן:**

| קובץ | חלק |
|---|---|
| `lessons/04-compose-networks-volumes/homework/docker-compose.yml` | א |
| `lessons/04-compose-networks-volumes/homework/init.sql` | א |
| `app/frontend/Dockerfile.multistage` | ב |
| `lessons/04-compose-networks-volumes/homework/answers.md` | ג |

> **למה בתיקייה `homework` ולא בתיקייה `docker`?** את `docker/docker-compose.yml` התחלתן לכתוב במחשב בכיתה, והוא שמור שם (בלי push). אם תכתבו קובץ באותו שם ובאותו מקום גם ב-GitHub, ב-`git pull` הבא יהיו שתי גרסאות שונות של אותו קובץ, ו-Git לא יידע איזו לבחור. בתיקייה נפרדת אין בעיה.

---

## חלק א — Compose לכל ShopList (חובה)

**המשימה:** לכתוב את כל `docker-compose.yml` של ShopList, עם ארבעת ה-services, בתיקייה `lessons/04-compose-networks-volumes/homework/`.

- **מי שלא סיימה בכיתה:** ממשיכה מאיפה שעצרה. המדריך [guide.md](guide.md), שלבים 6–7, עוזר.
- **מי שסיימה בכיתה:** כותבת שוב **מהזיכרון**, בלי להסתכל. רק בסוף משווה למדריך. ככה בודקים שזה באמת נכנס.

### הנתיבים: עולים שלוש תיקיות

הקובץ נמצא עמוק יותר מבכיתה, ולכן הנתיבים ל-`app` ארוכים יותר. נתיב ב-`context` הוא **יחסי למיקום של קובץ ה-compose**:

```
devops-2027-shoplist/                  ← ../../../  (שלוש פעמים למעלה)
├── app/backend                        ← ../../../app/backend
└── lessons/                           ← ../../
    └── 04-compose-networks-volumes/   ← ../
        └── homework/                  ← כאן הקובץ
```

### הדרישות

- [ ] ארבעה services, בשמות האלה בדיוק: `postgres`, `backend`, `stats`, `frontend`. השמות הם הכתובות, אז שגיאת כתיב אחת שוברת את הכול.
- [ ] **postgres:** `image: postgres:15-alpine`, שלושת משתני הסביבה (`POSTGRES_DB`, `POSTGRES_USER`, `POSTGRES_PASSWORD`), ו-`healthcheck` עם `pg_isready`.
- [ ] **postgres, volumes:** שתי שורות. אחת **named volume** בשם `postgres-data` לתיקייה `/var/lib/postgresql/data`. שנייה **bind mount** של `./init.sql` לתיקייה `/docker-entrypoint-initdb.d/init.sql`.
- [ ] **backend:** `build` עם `context: ../../../app/backend`, ארבעת משתני `DB_...`, ו-`depends_on` שמחכה ש-postgres יהיה `service_healthy`.
- [ ] **stats:** `build` עם context מתאים, ו-`BACKEND_URL` שמצביע על backend **לפי השם**.
- [ ] **frontend:** `build` עם context מתאים, ו-`ports: - "8080:80"`. זה ה-service היחיד עם `ports`.
- [ ] **בסוף הקובץ:** `volumes:` ברמה הראשית, עם `postgres-data:`.
- [ ] מעל כל service, הערה בעברית במשפט אחד: מה הוא, ועם מי הוא מדבר.

### init.sql

באותה תיקייה, קובץ `init.sql` עם טבלת המוצרים. התוכן: [guide.md](guide.md), שלב 4. כאן מותר להעתיק.

### איך עושים את זה ב-GitHub

1. ב-fork שלכן, לנווט ל-`lessons/04-compose-networks-volumes/homework/`.
2. **Add file** ואז **Create new file**, בשם `docker-compose.yml`.
3. לכתוב. ב-GitHub אין בדיקת שגיאות, אז לשים לב להזחות: **רווחים בלבד, שניים לכל רמה, לא Tab.**
4. **Commit changes**, עם ההודעה `homework 4: full compose`.
5. אותו דבר ל-`init.sql`.

---

## חלק ב — multi-stage ל-frontend (חובה)

### הרקע

בתיקייה `app/frontend` יש קובץ בשם `build.js`. זה סקריפט "בנייה" קטן ב-Node.js. הוא:

1. קורא את `index.html`, `app.js` ו-`style.css`.
2. מנקה את `app.js` מהערות ומשורות ריקות (כמו שכלים אמיתיים, Vite או webpack, מקטינים קוד).
3. מוסיף לתחתית הדף שורה עם זמן הבנייה: `Build: ...`.
4. כותב את התוצאה לתיקייה חדשה בשם `dist`.

כדי להריץ אותו צריך Node. כדי **להגיש** את התוצאה צריך רק Nginx. זה בדיוק המקרה של multi-stage: **מטבח** עם כל הכלים, ו**מגש הגשה** נקי עם העוגה בלבד.

### היעד

ליצור קובץ חדש: **`app/frontend/Dockerfile.multistage`** (D גדולה, נקודה, ו-multistage). לא לגעת ב-`Dockerfile` הרגיל. נשווה ביניהם בכיתה.

ב-GitHub: לנווט ל-`app/frontend`, **Add file** ואז **Create new file**.

### השלד

להעתיק את השלד, ולהשלים כל `____` לפי ההנחיות שמתחתיו:

```dockerfile
# שלב 1: בנייה (המטבח)
FROM ____ AS ____
WORKDIR /app
COPY ____ ____ ____ ____ ./
RUN ____

# שלב 2: הרצה (מגש ההגשה)
FROM ____
COPY --from=____ ____ /usr/share/nginx/html/
COPY ____ /etc/nginx/conf.d/default.conf
EXPOSE 80
```

### שלב 1: המטבח. מה נדרש

| שורה | מה להשלים | למה |
|---|---|---|
| `FROM ____ AS ____` | ה-image של Node שבו השתמשנו ל-stats, ואחרי `AS` את השם `build` | צריך Node כדי להריץ את `build.js`. `AS build` נותן לשלב שם, כדי שנוכל לפנות אליו בשלב 2 |
| `WORKDIR /app` | (מלא) | התיקייה שבה עובדים בתוך שלב הבנייה |
| `COPY ____ ____ ____ ____ ./` | ארבעת הקבצים ש-`build.js` צריך: הסקריפט עצמו, ושלושת קובצי האתר | כשמעתיקים כמה קבצים בשורה אחת, היעד חייב להסתיים ב-`/`. `./` פירושו "לתיקייה הנוכחית", כלומר `/app` |
| `RUN ____` | הפקודה שמריצה את סקריפט הבנייה עם Node | `RUN` רץ **בזמן הבנייה** של ה-image. בסוף השורה הזו קיימת התיקייה `/app/dist` |

**בסוף שלב 1** יש בתוך המטבח:

```
/app/
├── build.js, index.html, app.js, style.css   ← חומרי הגלם
└── dist/
    ├── index.html   ← עם שורת Build בתחתית
    ├── app.js       ← בלי הערות
    └── style.css
```

**מה לא כותבים בשלב 1:** לא `EXPOSE` ולא `CMD`. אף אחד לא מריץ את המטבח. הוא קיים רק כדי לבשל.

### שלב 2: מגש ההגשה. מה נדרש

| שורה | מה להשלים | למה |
|---|---|---|
| `FROM ____` | אותו image של Nginx כמו ב-Dockerfile הרגיל של ה-frontend | `FROM` שני מתחיל image **חדש ונקי**. שום דבר מהמטבח לא עובר לכאן, חוץ ממה שמעתיקים במפורש |
| `COPY --from=____ ____ /usr/share/nginx/html/` | אחרי `--from=` את שם השלב הראשון. אחריו את התיקייה שנוצרה בשלב 1 (נתיב מלא, שמתחיל ב-`/app`) | `--from` אומר: תעתיק **מהשלב ההוא**, לא מהמחשב. מעתיקים רק את `dist`, בלי `build.js` ובלי Node |
| `COPY ____ /etc/nginx/conf.d/default.conf` | קובץ ההגדרות של Nginx שכתבנו בכיתה | שימו לב: **בלי** `--from`. הקובץ הזה מגיע מהתיקייה `app/frontend` במחשב, כמו ב-Dockerfile הרגיל |
| `EXPOSE 80` | (מלא) | תיעוד בלבד, כמו תמיד |

**ואין `CMD`.** למה? כתבו את התשובה ב-`answers.md` (שאלה 5).

**בסוף שלב 2** יש ב-image הסופי: Nginx, שלושת הקבצים מ-`dist`, ו-`nginx.conf`. **אין** בו Node, אין `build.js`, ואין את הקבצים המקוריים.

### לחבר ל-compose של חלק א

ב-`docker-compose.yml` של חלק א, תחת `frontend` ← `build`, להוסיף שורה שאומרת ל-Compose באיזה Dockerfile להשתמש:

```yaml
  frontend:
    build:
      context: ../../../app/frontend
      dockerfile: Dockerfile.multistage
```

הנתיב ב-`dockerfile` יחסי ל-`context`, כלומר לתיקייה `app/frontend`. לכן מספיק השם.

### טעויות נפוצות

| טעות | מה יקרה |
|---|---|
| שכחתן `AS build` בשלב 1 | `--from=build` לא ימצא שלב בשם הזה, וה-build ייכשל |
| `COPY --from=build dist/ ...` בלי `/app` | הנתיב בשלב 1 הוא `/app/dist`, ולא יימצא |
| שכחתן את `COPY nginx.conf` בשלב 2 | בדפדפן יופיע `Welcome to nginx!` במקום ShopList |
| כתבתן `--from=build` גם על `nginx.conf` | `nginx.conf` לא הועתק לשלב 1, ולכן הוא לא נמצא שם |
| קראתן לקובץ `dockerfile.multistage` (d קטנה) | בכיתה, ב-Windows, זה יעבוד. ב-Linux ובשרתים לא. לשמור על D גדולה |

**Commit:** `homework 4: multi-stage frontend`.

---

## חלק ג — שאלות (חובה)

ליצור בתיקייה `lessons/04-compose-networks-volumes/homework/` קובץ `answers.md` ולענות במשפט או שניים:

1. בקובץ שלכן, ל-backend אין `ports`. איך stats מגיע אליו? ואיך הדפדפן מגיע אליו?
2. עושים `docker compose down` ואחר כך `up`. המוצרים נשארו? ומה אם עושים `down -v`? הסבירו לפי סוג ה-volume.
3. השורה `./init.sql:/docker-entrypoint-initdb.d/init.sql`: איזה סוג volume זה, ואיך יודעים? מה יקרה אם תשנו את `init.sql` אחרי שהמסד כבר נוצר?
4. הקובץ שלכן נמצא בתיקייה `homework`. מה יהיה שם הרשת ש-Compose ייצור, ומה שם ה-volume? (רמז: שם התיקייה.)
5. ב-`Dockerfile.multistage`: מה יש ב-image הסופי ומה אין? למה אין `CMD`? ולמה ל-stats לא צריך multi-stage?

---

## בונוס

**סיסמאות מחוץ לקובץ.** הסיסמה `shoppass` כתובה פעמיים בקובץ, והקובץ ב-Git. להעביר אותה לקובץ `.env` באותה תיקייה:

```
POSTGRES_PASSWORD=shoppass
```

ובקובץ ה-compose, בשני המקומות: `${POSTGRES_PASSWORD}`. **את `.env` עצמו לא מעלים ל-GitHub.** ב-`answers.md`, להסביר במשפט: איך מוודאים ש-`.env` לא ייכנס ל-Git?

---

## בתחילת שיעור 5, בכיתה

1. **ב-GitHub:** Sync fork ואז Update branch.
2. **במחשב:**

```shell
cd devops-2027-shoplist
git pull --no-rebase --no-edit
```

> `--no-rebase --no-edit`: במחשב בכיתה יש commits שלא עלו ל-GitHub, וב-GitHub יש את שיעורי הבית. Git צריך לחבר ביניהם. שני הדגלים אומרים לו לחבר בלי לשאול ובלי לפתוח עורך.

3. **להשוות גדלים:**

```shell
docker build -t shoplist-frontend:single app/frontend
docker build -f app/frontend/Dockerfile.multistage -t shoplist-frontend:multi app/frontend
docker build -f app/frontend/Dockerfile.multistage --target build -t shoplist-frontend:kitchen app/frontend
docker images shoplist-frontend
```

- `--target build`: לבנות **רק את המטבח**, כדי לראות כמה הוא שוקל.
- `kitchen` גדול בהרבה מ-`multi`. ו-`multi` כמעט בגודל של `single`: את Node השארנו במטבח.

```shell
docker run --rm shoplist-frontend:kitchen node --version   # v24...
docker run --rm shoplist-frontend:multi node --version     # node: not found
```

4. **להריץ את ה-compose שלכן:**

```shell
cd docker
docker compose down
cd ../lessons/04-compose-networks-volumes/homework
docker compose up -d --build
docker compose ps
```

ב-`http://localhost:8080`, בתחתית הדף: `Build: ...` עם התאריך. זה הסימן שה-frontend נבנה מה-multi-stage.

```shell
docker compose down
```
