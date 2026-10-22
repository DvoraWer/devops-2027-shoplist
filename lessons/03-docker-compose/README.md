# שיעור 3: Docker Compose, Nginx ושמירת נתונים

## מה לומדות בשיעור

- Nginx כ-reverse proxy: כתובת אחת לדפדפן, וכל בקשה לסרוויס הנכון
- Dockerfile ל-frontend
- Docker Compose: כל ארבעת הסרוויסים בקובץ אחד ובפקודה אחת
- איך סרוויסים מוצאים זה את זה לפי השם
- volumes, ומה עושה `down -v`
- healthcheck: לחכות שמסד הנתונים באמת מוכן
- multi-stage build: image סופי קטן, בלי כלי הבנייה

## התרגילים

1. מה קורה ל-frontend לבד?
2. להציץ למסד הנתונים עם `psql`
3. בלי healthcheck
4. סיסמאות מחוץ לקובץ, עם `.env`
5. multi-stage: Dockerfile שבונה עם Node ומריץ עם Nginx
6. בונוס: מוצר אחד לפי מספר
7. תרגול פקודות: [commands-practice.md](commands-practice.md)

## שיעורי בית

[homework.md](homework.md): Compose ל-greeter בשתי שפות, והגשת הבוחן. ישירות ב-GitHub, בלי התקנות בבית.

## הקבצים בתיקייה

| קובץ | מה יש בו |
|---|---|
| [guide.md](guide.md) | מדריך שלב אחר שלב |
| [exercises.md](exercises.md) | תרגילים |
| [compose-reference.md](compose-reference.md) | מדריך עזר: פקודות Compose, YAML ו-nginx.conf |
| [commands-practice.md](commands-practice.md) | תרגול פקודות: על דף ובמחשב |
| [quiz.md](quiz.md) | **בוחן להגשה** על שיעור 3 |
| [homework.md](homework.md) | **שיעורי בית** |
