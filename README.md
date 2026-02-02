1.  Архитектура компонента

```
QuestionCard
├── QuestionStem (TipTapRenderer + KaTeX)
├── AnswerOptions (RadioGroup для single-select)
├── ActionBar
│ ├── CheckButton
│ └── NextButton (появляется после проверки)
└── Explanation (conditional render)
```

Хранение состояния:

selectedAnswer, isChecked — локальное состояние в QuestionCard (useState)

isSubmitting — локальное состояние для отслеживания отправки

questionData, explanationData — глобальное состояние (store/props)

Сброс при смене questionId:

selectedAnswer = null
isChecked = false
Закрываем explanation (через сброс локального состояния)

При быстрых кликах:

CheckButton получает дебаунс 0.3 секунды

Показываем спиннер загрузки на кнопке isSubmitting

Не даем взаимодействовать со всеми интерактивными элементами во время загрузки

Отменяем прошлый запрос с помощью AbortController

После того как сервер ответит, обновляем состояние только для текущего questionId

2. Псевдокод логики

```
2.1 Инициализация состояния
state = {
selectedAnswer: null,
isChecked: false,
isSubmitting: false
}

2.2 Выбор ответа
handleSelectAnswer(answerId):
if !isChecked AND !isSubmitting:
setSelectedAnswer(answerId)

// Проверка ответа
handleCheckAnswer():
if !selectedAnswer OR isSubmitting:
return

setSubmitting(true)

try:
// Отправка на сервер
response = await api.checkAnswer({
questionId: currentQuestionId,
answer: selectedAnswer
})

    // Обновление состояния
    setChecked(true)
    // Сохраняем explanation в глобальный store
    store.saveExplanation(response.explanation)

catch error:
showError("Ошибка при проверке")
finally:
setSubmitting(false)

// Смена вопроса
handleQuestionChange(newQuestionId):
// Полный сброс локального состояния
setSelectedAnswer(null)
setChecked(false)
setSubmitting(false)
// Скрываем explanation

// Условия disabled
checkButtonDisabled = !selectedAnswer OR isChecked OR isSubmitting
answerOptionsDisabled = isChecked OR isSubmitting
nextButtonVisible = isChecked

// Рендер explanation только если:
showExplanation = isChecked AND explanationExists
```

3. Edge cases и UX

1 - Explanation отсутствует

Показываем информационный блок: "Для этого вопроса нет объяснения"

Серый фон, иконка ℹ️, текст курсивом

Высота блока фиксирована для сохранения layout

2 - В stem только формулы

Минимальная высота контейнера: 120px

Вертикальное центрирование через flexbox

Увеличенный размер формул (+20%)

3 - Очень длинный текст

max-height: 400px с overflow-y: auto

Градиент внизу при обрезке

Кнопка "Развернуть" (раскрывает на всю высоту)

4 - KaTeX упал с ошибкой

Фолбэк: показываем оригинальный LaTeX-код в < code>

Сообщение: "Не удалось отобразить формулу"

Кнопка "Скопировать код формулы"

5 - Пользователь меняет ответ после check

Вариант 5.1 --> Отключаем выбор после проверки

Вариант 5.2 --> Кнопка "Изменить ответ" → сбрасывает isChecked

6 - Пользователь в demo режиме

```
<DemoOverlay>
<BlurredContent>
<Explanation content={explanation} />
</BlurredContent>

  <DemoMessage>
    <LockIcon />
    <h4>Объяснение доступно в Pro-версии</h4>
    <p>Узнайте, почему ответ правильный или неправильный</p>
    <UpgradeButton onClick={openPricing}>
      Перейти к тарифам
    </UpgradeButton>
  </DemoMessage>
</DemoOverlay>
```
