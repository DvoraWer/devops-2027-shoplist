רכיב	איפה הוא רץ	מה הוא עושה, במשפט אחד	אנלוגיה
API server	הנהלה	נקודת הכניסה היחידה ל-cluster: מקבל כל בקשה, בודק שהיא תקינה ומותרת, ורושם אותה	מוקד ההזמנות: כל הזמנה עוברת דרכו, וכל העובדים מתעדכנים רק דרכו
etcd	הנהלה	שומר את כל המידע על ה-cluster: מה אמור לרוץ ומה רץ בפועל	ספר ההזמנות: מה הוזמן ומה כבר מוכן
scheduler	הנהלה	בוחר לכל pod חדש על איזה node ירוץ, לפי המקום הפנוי	הסדרנית שמחליטה איזה מטבח יאפה כל עוגה
controller manager	הנהלה	משווה כל הזמן בין המצב הרצוי למצב בפועל ומתקן פערים	המפקחת שבודקת שהוויטרינה מלאה, ומזמינה עוגה חדשה כשחסרה
kubelet	כל node	מקבלת מה שובץ ל-node שלה, דואגת שזה ירוץ ומדווחת שה-node חי	אחראית המשמרת בכל מטבח
container runtime	כל node	מריץ בפועל את ה-containers מתוך ה-images	התנור שבו העוגה באמת נאפית
kube-proxy	כל node	מעביר בקשה שמגיעה לשם של Service לאחד ה-pods המתאימים	השליחה שמביאה כל הזמנה לדלפק הנכון
coredns	רץ כ-pods ב-namespace kube-system	מתרגם שם של Service, כמו greeter, לכתובת אמיתית	ספר הטלפונים הפנימי: מחפשים לפי שם ומקבלים את המספר

**1. scale**
הפקודה נשלחת ל-API server, שבודק אותה ומעדכן ב-etcd שהמצב הרצוי הוא 4 pods. ה-controller manager רואה שקיימים רק 2 מתוך 4, ויוצר 2 pods חדשים. ה-scheduler משבץ כל pod חדש ל-node שיש בו מקום פנוי. ה-kubelet בכל node רואה מה שובץ אליה, ה-container runtime מריץ את ה-containers, וה-kubelet מדווחת ל-API server שה-pods במצב Running.

**2. שלוש בלילה**
כשה-container קורס, ה-kubelet שעל ה-node שמה לב ומפעילה אותו מחדש בתוך אותו pod, בלי שההנהלה מעורבת. אם מוחקים את כל ה-pod, ה-container נמחק יחד איתו. אבל ה-controller manager רואה שיש פחות pods ממה שרשום ב-Deployment, ויוצר pod חדש. ה-scheduler משבץ אותו, וה-kubelet מריצה אותו.

**3. node נכבה**
ה-kubelet של node 2 מפסיקה לדווח, וההנהלה מסמנת אותו NotReady. ה-controller manager יוצר pods חלופיים, וה-scheduler משבץ אותם ל-nodes 1 ו-3 שעדיין פעילים. ב-Docker Desktop יש node אחד בלבד, ועליו רצה גם ההנהלה עצמה. לכן אם המחשב נכבה, אין לאן להעביר את ה-pods, וכל ה-cluster מפסיק לעבוד.

**4. מהכיתה לענן**
הגדרת `imagePullPolicy: Never` אומרת להשתמש רק ב-image שנמצא על המחשב. אבל בשרתי EKS ה-image לא קיים, ולכן ה-pods ייכשלו. צריך להעלות את ה-image ל-registry כמו Docker Hub או ECR, לעדכן בקובץ את הכתובת שלו, ולשנות את ההגדרה ל-`Always` או ל-`IfNotPresent`. בנוסף, NodePort פותח גישה רק דרך פורט על ה-node, ולא נותן כתובת ציבורית למשתמשים מהאינטרנט. בענן מחליפים אותו ב-LoadBalancer.

**5. pod בודד**
pod שנוצר עם `kubectl run` עומד לבד: אין עליו הזמנה קבועה ב-etcd, ולכן כשהוא נמחק אף אחד לא מחזיר אותו. ה-pod של web שייך ל-Deployment. ה-controller manager רואה שחסר pod, ויוצר pod חדש במקומו (עם שם חדש). כדי ש-stats יחזור תמיד, ניצור לו Deployment, כי לא מריצים pod לבד.
