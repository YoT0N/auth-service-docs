# UI-документація (Storybook) — Auth Service

**Версія:** v1.0  
**Останнє оновлення:** 2025  
**Автор:** Ільків Богдан

> Storybook використовується для ізольованої розробки та документування UI-компонентів фронтенд-частини Auth Service. Кожен компонент описується через Stories — набір прикладів з різними станами.

---

## Встановлення та запуск

```bash
# Встановлення Storybook (якщо не встановлено)
npx storybook@latest init

# Запуск у режимі розробки
npm run storybook

# Збірка статичного сайту
npm run build-storybook
```

Storybook доступний локально за адресою: `http://localhost:6006`

---

## Компонент 1: Button

### Призначення

Кнопка — базовий інтерактивний елемент, що використовується у формах автентифікації (Login, Register). Підтримує різні варіанти стану для відображення процесу та помилок.

### Stories

#### Button.stories.jsx

```jsx
import { Button } from './Button';

export default {
  title: 'Auth/Button',
  component: Button,
  tags: ['autodocs'],
  argTypes: {
    variant: {
      control: 'select',
      options: ['primary', 'secondary', 'danger'],
      description: 'Візуальний варіант кнопки',
    },
    disabled: {
      control: 'boolean',
      description: 'Чи заблокована кнопка',
    },
    loading: {
      control: 'boolean',
      description: 'Стан завантаження (показує спінер)',
    },
    label: {
      control: 'text',
      description: 'Текст кнопки',
    },
  },
};

// ──────────────────────────────────────────
// 1. Default — базовий стан
// ──────────────────────────────────────────
export const Default = {
  args: {
    label: 'Увійти',
    variant: 'primary',
    disabled: false,
    loading: false,
  },
};

// ──────────────────────────────────────────
// 2. Disabled — кнопка заблокована
// Використовується: форма не заповнена або дія недоступна
// ──────────────────────────────────────────
export const Disabled = {
  args: {
    label: 'Увійти',
    variant: 'primary',
    disabled: true,
    loading: false,
  },
};

// ──────────────────────────────────────────
// 3. Loading — очікування відповіді від сервера
// Використовується: після відправки форми, під час запиту
// ──────────────────────────────────────────
export const Loading = {
  args: {
    label: 'Входимо...',
    variant: 'primary',
    disabled: true,
    loading: true,
  },
};

// ──────────────────────────────────────────
// 4. Danger — небезпечна дія (наприклад, вихід)
// ──────────────────────────────────────────
export const Danger = {
  args: {
    label: 'Вийти',
    variant: 'danger',
    disabled: false,
    loading: false,
  },
};
```

### Реалізація компонента

```jsx
// Button.jsx
export const Button = ({ label, variant = 'primary', disabled = false, loading = false, onClick }) => {
  const baseStyles = 'px-4 py-2 rounded font-medium transition-all duration-200 flex items-center gap-2';
  
  const variants = {
    primary: 'bg-blue-600 text-white hover:bg-blue-700 disabled:bg-blue-300',
    secondary: 'bg-gray-200 text-gray-800 hover:bg-gray-300 disabled:bg-gray-100',
    danger: 'bg-red-600 text-white hover:bg-red-700 disabled:bg-red-300',
  };

  return (
    <button
      className={`${baseStyles} ${variants[variant]}`}
      disabled={disabled || loading}
      onClick={onClick}
    >
      {loading && <span className="spinner" aria-hidden="true" />}
      {label}
    </button>
  );
};
```

---

## Компонент 2: InputField

### Призначення

Поле вводу з підтримкою підказки, помилки та типу password (з кнопкою показу пароля). Використовується у формах Login та Register.

### Stories

#### InputField.stories.jsx

```jsx
import { InputField } from './InputField';

export default {
  title: 'Auth/InputField',
  component: InputField,
  tags: ['autodocs'],
  argTypes: {
    type: {
      control: 'select',
      options: ['text', 'email', 'password'],
      description: 'Тип поля вводу',
    },
    error: {
      control: 'text',
      description: 'Текст помилки (порожній = без помилки)',
    },
    disabled: {
      control: 'boolean',
      description: 'Чи заблоковане поле',
    },
  },
};

// ──────────────────────────────────────────
// 1. Default — базовий стан (email)
// ──────────────────────────────────────────
export const Default = {
  args: {
    label: 'Email',
    type: 'email',
    placeholder: 'user@example.com',
    error: '',
    disabled: false,
  },
};

// ──────────────────────────────────────────
// 2. Error — поле з помилкою валідації
// Використовується: після відправки форми з невалідними даними
// ──────────────────────────────────────────
export const WithError = {
  name: 'Error state',
  args: {
    label: 'Email',
    type: 'email',
    placeholder: 'user@example.com',
    value: 'not-an-email',
    error: 'Введіть коректну email-адресу',
    disabled: false,
  },
};

// ──────────────────────────────────────────
// 3. Password — поле з паролем і перемикачем видимості
// ──────────────────────────────────────────
export const Password = {
  args: {
    label: 'Пароль',
    type: 'password',
    placeholder: '••••••••',
    error: '',
    disabled: false,
  },
};

// ──────────────────────────────────────────
// 4. Disabled — заблоковане поле
// ──────────────────────────────────────────
export const Disabled = {
  args: {
    label: 'Email',
    type: 'email',
    placeholder: 'user@example.com',
    value: 'user@example.com',
    error: '',
    disabled: true,
  },
};
```

### Реалізація компонента

```jsx
// InputField.jsx
import { useState } from 'react';

export const InputField = ({ label, type = 'text', placeholder, value, error, disabled, onChange }) => {
  const [showPassword, setShowPassword] = useState(false);
  const inputType = type === 'password' && showPassword ? 'text' : type;

  return (
    <div className="flex flex-col gap-1">
      <label className="text-sm font-medium text-gray-700">{label}</label>
      <div className="relative">
        <input
          type={inputType}
          placeholder={placeholder}
          value={value}
          disabled={disabled}
          onChange={onChange}
          className={`
            w-full px-3 py-2 border rounded-md text-sm
            ${error ? 'border-red-500 bg-red-50' : 'border-gray-300'}
            ${disabled ? 'bg-gray-100 text-gray-400 cursor-not-allowed' : ''}
            focus:outline-none focus:ring-2 focus:ring-blue-500
          `}
        />
        {type === 'password' && (
          <button
            type="button"
            onClick={() => setShowPassword(!showPassword)}
            className="absolute right-2 top-1/2 -translate-y-1/2 text-gray-400 hover:text-gray-600"
          >
            {showPassword ? '🙈' : '👁️'}
          </button>
        )}
      </div>
      {error && <span className="text-xs text-red-600">{error}</span>}
    </div>
  );
};
```

---

## Структура Storybook у проєкті

```
src/
├── stories/
│   ├── Button.jsx              ← компонент
│   ├── Button.stories.jsx      ← stories
│   ├── InputField.jsx
│   └── InputField.stories.jsx
└── .storybook/
    ├── main.js                 ← конфігурація Storybook
    └── preview.js              ← глобальні декоратори
```

---

## Матриця станів компонентів

| Компонент | Default | Disabled | Error / Loading | Додаткові стани |
|-----------|---------|----------|-----------------|-----------------|
| Button | ✓ | ✓ | ✓ (Loading) | Danger |
| InputField | ✓ | ✓ | ✓ (Error) | Password |

---

## Інтеграція зі Storybook у CI/CD

При кожному push до репозиторію виконується:

```bash
npm run build-storybook -- --output-dir site/storybook
```

Результат публікується на GitHub Pages за адресою:
```
https://<org>.github.io/auth-service/storybook/
```
