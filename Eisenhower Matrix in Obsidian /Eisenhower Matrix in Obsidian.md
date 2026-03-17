# Eisenhower Matrix in Obsidian - Таблица эзенхаура в обсидиане 🚀️

**Таблица Эйзенхауэра (Eisenhower Matrix)** — это инструмент приоритизации задач, основанный на двух критериях:

* **важность**
* **срочность**

Она помогает быстро определить, какие задачи требуют немедленного внимания, какие нужно планировать, какие делегировать, а какие — исключить.

|  тег  | описание                    | Действие             |
| :--------: | :------------------------------------ | ------------------------------ |
|  #raf_i  | Срочно и важно          | Делать сейчас    |
| #raf_ii | Несрочно, но важно   | Планировать       |
| #raf_iii | Срочно, но не важно  | Делегировать     |
| #raf_vi | Несрочно и не важно | Минимизировать |

## Как именно пользоваться в обсидиане?

Например написали какую статью или начали работать над каким то проектом, и нам необходимо в дальнейшем проработать какие то аспекты. Нам необходимо в конце статьи добавить список задач с тегом #raf_i - #raf_vi и рядом написать задачу. Списки задач можно добавлть в любых заметках.

### Пример

[ ] #raf_vi  если будет время, необходимо приобретсти вариатор
[ ] #raf_i Необходимо, подписать договор
[ ] ...

## Необходимый код для запуска

1. Нобходимо скачать плагин **Dataview**. Необходимо войти в настройки -> стронние плагины -> обхор -> ищем данный плагин, после скачиваем и устанавливаем. В настройках плагина, включаем javascript
2. Переходим в заметку где будет отображаться таблица Эзенхаура и туда вставляем данный код:

````
```dataviewjs
(() => {

const ANIMATION = "balls"; // ← выбери: "dots" или "balls"

const BLOCK_TAGS = {
    raf_i: "I",
    raf_ii: "II",
    raf_iii: "III",
    raf_vi: "IV"
};

const meta = {
    I: ["Срочно и важно", "Сделать сейчас"],
    II: ["Несрочно, но важно", "Запланировать"],
    III: ["Срочно, но не важно", "Делегировать"],
    IV: ["Несрочно и не важно", "Минимизировать"]
};

function extractTags(text, taskTags) {
    const tags = new Set();
    if (taskTags) for (const t of taskTags) tags.add(t.replace("#","").trim().toLowerCase());
    for (const part of text.split(" ")) if (part.startsWith("#")) tags.add(part.slice(1).trim().toLowerCase());
    return [...tags];
}

function cleanText(text) {
    return text.split(" ").filter(p => !p.startsWith("#")).join(" ").trim();
}

const blocks = { I: [], II: [], III: [], IV: [] };
const done = [];

for (const page of dv.pages()) {
    if (!page.file?.tasks) continue;

    for (const t of page.file.tasks) {
        const raw = t.text ?? "";
        const text = cleanText(raw);

        const completed = t.completed === true || raw.trim().toLowerCase().startsWith("[x]");
        const tags = extractTags(raw, t.tags);
        if (!tags.length) continue;

        let blockKey = null;
        for (const tag of tags) if (BLOCK_TAGS[tag]) blockKey = BLOCK_TAGS[tag];
        if (!blockKey) continue;

        const task = {
            text,
            completed,
            due: t.due ? dv.date(t.due) : null,
            path: page.file.path,
            line: t.line,
            filename: page.file.name
        };

        (completed ? done : blocks[blockKey]).push(task);
    }
}

function sortTasks(list) {
    list.sort((a,b)=>{
        if (a.due && b.due) return a.due - b.due;
        if (a.due) return -1;
        if (b.due) return 1;
        return a.text.localeCompare(b.text);
    });
}

Object.values(blocks).forEach(sortTasks);
sortTasks(done);

const root = dv.container.createEl("div", { cls: "eh-grid" });

function renderTask(container, task) {
    const row = container.createEl("div", { cls: "eh-task" });
    const left = row.createEl("div", { cls: "eh-left" });

    left.createEl("span", { text: task.text });

    if (task.due) {
        const due = row.createEl("span", {
            text: task.due.toFormat("yyyy-MM-dd"),
            cls: "eh-due"
        });
        if (task.due < dv.date("today")) due.addClass("eh-overdue");
    }

    const source = row.createEl("div", { cls: "eh-source" });

    const link = source.createEl("a", {
        cls: "internal-link eh-note-link"
    });

    link.setAttr("data-href", task.path);
    link.setAttr("href", task.path);

    const anim = link.createEl("div", { cls: ANIMATION });

    if (ANIMATION === "dots") {
        for (let i = 0; i < 9; i++) anim.createEl("div");
    } else if (ANIMATION === "balls") {
        anim.createEl("div");
        anim.createEl("div");
        anim.createEl("div");
    }
}

function renderBlock(key) {
    const box = root.createEl("div", { cls: `eh-box eh-${key}` });
    const header = box.createEl("div", { cls: "eh-header" });

    header.createEl("div", { text: `${meta[key][0]} (${blocks[key].length})`, cls: "eh-title" });
    header.createEl("div", { text: meta[key][1], cls: "eh-desc" });

    const body = box.createEl("div", { cls: "eh-body" });

    if (!blocks[key].length) {
        body.createEl("div", { text: "Нет задач", cls: "eh-empty" });
        return;
    }

    for (const t of blocks[key]) renderTask(body, t);
}

["I","II","III","IV"].forEach(renderBlock);

if (done.length) {
    const doneBox = root.createEl("div", { cls: "eh-box eh-done" });
    doneBox.createEl("div", { text: `Выполненные задачи (${done.length})`, cls: "eh-title" });
    const doneBody = doneBox.createEl("div", { cls: "eh-body" });
    for (const t of done) renderTask(doneBody, t);
}

})();

```
````

## Добавляем css файл

1. На данный момент, сделаем нашу таблицу эзенхаура более менее красивым. Нам необходимо добавить добавить файл  со стилями. Переходим в настройки -> оформление -> и снизу находим "фрагменты css кода".
2. Нажимаем на иконку папка и откроется папка.
3. В данной папке создаем файл с расширением `.css`, данному файлу можно присвоить любое имя. Я назвал ее `Eisenhower.css`
4. Данный файл открываем с помошью [Notepad++](https://notepad-plus-plus.org/downloads/) и копируем следующий фрагмент кода и сохраняем. Не забываем, включить данный стиль, в оформлении.

````
/* =======================
   ОСНОВНОЙ КОНТЕЙНЕР
======================= */
.eh-grid {
  display: flex;
  flex-direction: column;
  gap: 12px;
  background: #08080d;
  padding: 16px;
  font-family: 'Inter', sans-serif;
  min-height: 100vh;
  font-size: 0.9rem;
}

/* =======================
   КВАДРАНТЫ / КАРТОЧКИ С ЦВЕТОМ
======================= */
.eh-box {
  border-radius: 14px;
  padding: 12px 16px;
  border-left: 3px solid;
  transition: all 0.2s ease;
  background: #111;
}

/* Цвета карточек по категориям */
.eh-I {
  border-left-color: #00fff0;
  background: linear-gradient(90deg, #0a0a0f, #081f22);
}
.eh-II {
  border-left-color: #3399ff;
  background: linear-gradient(90deg, #0a0a0f, #081a3b);
}
.eh-III {
  border-left-color: #33ff99;
  background: linear-gradient(90deg, #0a0a0f, #08261a);
}
.eh-IV {
  border-left-color: #ffcc33;
  background: linear-gradient(90deg, #0a0a0f, #332600);
}
.eh-done {
  border-left-color: #66ff66;
  background: linear-gradient(90deg, #0a0a0f, #193219);
}

.eh-box:hover {
  transform: translateY(-2px);
  box-shadow: inset 0 0 5px rgba(255,255,255,0.05);
}

/* =======================
   ЗАГОЛОВКИ И ОПИСАНИЕ
======================= */
.eh-header {
  margin-bottom: 6px;
}

.eh-title {
  font-weight: 600;
  letter-spacing: 1px;
  font-size: 1.2rem;
}

.eh-I .eh-title { color: #00fff0; }
.eh-II .eh-title { color: #3399ff; }
.eh-III .eh-title { color: #33ff99; }
.eh-IV .eh-title { color: #ffcc33; }
.eh-done .eh-title { color: #66ff66; }

.eh-desc {
  font-size: 0.75rem;
  color: rgba(255,255,255,0.6);
}

/* =======================
   КОНТЕЙНЕР ЗАДАЧ
======================= */
.eh-body {
  display: flex;
  flex-direction: column;
  gap: 6px;
}

.eh-task {
  display: flex;
  align-items: center;
  justify-content: space-between;
  border-radius: 12px;
  padding: 6px 10px;
  border-left: 3px solid;
  transition: all 0.2s ease;
  background: #111;
}

/* Цвет задач по категории */
.eh-I .eh-task { border-left-color: #00fff0; background: #081f22; }
.eh-II .eh-task { border-left-color: #3399ff; background: #081a3b; }
.eh-III .eh-task { border-left-color: #33ff99; background: #08261a; }
.eh-IV .eh-task { border-left-color: #ffcc33; background: #332600; }
.eh-done .eh-task { border-left-color: #66ff66; background: #193219; }

.eh-task:hover {
  transform: translateX(2px);
  box-shadow: inset 0 0 3px rgba(255,255,255,0.05);
}

.eh-left {
  display: flex;
  align-items: center;
  gap: 5px;
}

.eh-due {
  font-size: 0.7rem;
  color: rgba(255,255,255,0.5);
}

.eh-overdue {
  font-weight: 600;
  color: #ff4444;
}

/* =======================
   ПУСТЫЕ БЛОКИ
======================= */
.eh-empty {
  font-style: italic;
  color: rgba(255,255,255,0.3);
  animation: pulse 1.2s infinite alternate;
}

@keyframes pulse {
  0% { opacity: 0.4; }
  100% { opacity: 0.7; }
}

/* =======================
   ССЫЛКИ В ЗАДАЧАХ
======================= */
.eh-task a.internal-link {
  text-decoration: none;
  transition: color 0.2s ease;
}

.eh-I .eh-task a.internal-link { color: #00fff0; }
.eh-II .eh-task a.internal-link { color: #3399ff; }
.eh-III .eh-task a.internal-link { color: #33ff99; }
.eh-IV .eh-task a.internal-link { color: #ffcc33; }
.eh-done .eh-task a.internal-link { color: #66ff66; }

.eh-task a.internal-link:hover {
  filter: brightness(1.3);
}

/* =======================
   DOTS 3x3 АНИМАЦИЯ
======================= */
.dots {
  display: grid;
  grid-template-columns: repeat(3, 5px);
  gap: 3px;
}

.dots > div {
  width: 5px;
  height: 5px;
  border-radius: 50%;
  animation: simple-pulse 1s infinite alternate;
}

.eh-I .dots > div { background: #00fff0; }
.eh-II .dots > div { background: #3399ff; }
.eh-III .dots > div { background: #33ff99; }
.eh-IV .dots > div { background: #ffcc33; }
.eh-done .dots > div { background: #66ff66; }

.dots > div:nth-of-type(1) { animation-delay: 0s; }
.dots > div:nth-of-type(2) { animation-delay: 0.05s; }
.dots > div:nth-of-type(3) { animation-delay: 0.1s; }

@keyframes simple-pulse {
  0% { transform: scale(0.6); opacity: 0.4; }
  100% { transform: scale(0.9); opacity: 0.7; }
}

/* =======================
   BALLS АНИМАЦИЯ
======================= */
.balls {
  display: flex;
  gap: 5px;
}

.balls div {
  width: 6px;
  height: 6px;
  border-radius: 50%;
  animation: simple-swing 0.5s infinite alternate;
}

.eh-I .balls div { background: #00fff0; }
.eh-II .balls div { background: #3399ff; }
.eh-III .balls div { background: #33ff99; }
.eh-IV .balls div { background: #ffcc33; }
.eh-done .balls div { background: #66ff66; }

.balls div:nth-of-type(1) { animation-delay: 0s; }
.balls div:nth-of-type(2) { animation-delay: 0.05s; }
.balls div:nth-of-type(3) { animation-delay: 0.1s; }

@keyframes simple-swing {
  0% { transform: translateY(0); }
  100% { transform: translateY(-2px); }
}
````

