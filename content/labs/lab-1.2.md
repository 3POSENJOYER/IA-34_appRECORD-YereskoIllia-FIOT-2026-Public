## Тема

«ВСТАНОВЛЕННЯ ТА НАЛАШТУВАННЯ СЕРЕДОВИЩА NODE.JS ОСНОВИ РОБОТИ З EXPRESS.JS»

## Мета

Ознайомитися з принципами роботи HTTP-серверів.
Вивчити основи створення веб-серверів на Node.js.
Ознайомитися з архітектурою REST API.
Навчитися створювати маршрути (routes) для обробки HTTP-запитів;

---

## Завдання 1

Створити Node.js проєкт та встановити бібліотеку Express.js.
![Скрін 1](/assets/labs/lab.1.2/str.jpg)

---

## Завдання 2

Створити HTTP-сервер, який повертає повідомлення:
![Скрін 1](/assets/labs/lab.1.2/hello.jpg)

---

## Завдання 3

Створити маршрут:
який повертає список студентів у форматі JSON.
![Скрін 1](/assets/labs/lab.1.2/get.jpg)

---

## Завдання 4

Створити маршрут:
для додавання нового студента.

```bash
app.post('/students', (req, res) => {
  const { id, name, group } = req.body;

  if (id === undefined || !name || !group) {
    return res.status(400).json({
      message: 'Потрібно передати поля: id, name, group'
    });
  }

  const existingStudent = students.find(student => student.id === id);
  if (existingStudent) {
    return res.status(400).json({
      message: 'Студент з таким id вже існує'
    });
  }

  const newStudent = { id, name, group };
  students.push(newStudent);

  res.status(201).json({
    message: 'Студента додано успішно',
    student: newStudent
  });
});
```

---

## Завдання 5

Створити маршрути:
для оновлення та видалення студента.
Оновлення студента

```bash
app.put('/students/:id', (req, res) => {
  const studentId = Number(req.params.id);
  const { name, group } = req.body;

  const studentIndex = students.findIndex(student => student.id === studentId);

  if (studentIndex === -1) {
    return res.status(404).json({
      message: 'Студента не знайдено'
    });
  }

  if (!name && !group) {
    return res.status(400).json({
      message: 'Передай хоча б одне поле для оновлення: name або group'
    });
  }

  if (name) {
    students[studentIndex].name = name;
  }

  if (group) {
    students[studentIndex].group = group;
  }

  res.json({
    message: 'Студента оновлено успішно',
    student: students[studentIndex]
  });
});
```

Видалення студента

```bash
app.delete('/students/:id', (req, res) => {
  const studentId = Number(req.params.id);

  const studentIndex = students.findIndex(student => student.id === studentId);

  if (studentIndex === -1) {
    return res.status(404).json({
      message: 'Студента не знайдено'
    });
  }

  const deletedStudent = students.splice(studentIndex, 1);

  res.json({
    message: 'Студента видалено успішно',
    student: deletedStudent[0]
  });
});
```

## Висновки

Проєкт забезпечує повноцінну CRUD (Create, Read, Update, Delete) операційну систему для управління студентами через API на базі Node.js та Express.js. Всі маршрути обробляються сервером, що дозволяє взаємодіяти з даними через HTTP-запити. Це є основою для побудови більших систем з управління даними.
