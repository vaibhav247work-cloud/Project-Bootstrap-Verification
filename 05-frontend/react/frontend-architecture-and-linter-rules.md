# Frontend Architecture and Linter Enforcement Standard

## Goal

Automate frontend architecture boundaries, file size limits, and code hygiene through ESLint and TypeScript compiler rules. Violations must fail in CI before code can be merged.

## 1. File Size Limits Configuration

Configure ESLint's `max-lines` rule in `.eslintrc.cjs` or `eslint.config.js`:

```javascript
module.exports = {
  rules: {
    // General files (hooks, utilities, schemas): max 300 lines
    'max-lines': [
      'error',
      {
        max: 300,
        skipBlankLines: true,
        skipComments: true,
      },
    ],
  },
  overrides: [
    {
      // TSX Component files: max 400 lines (absolute ceiling 600)
      files: ['*.tsx'],
      rules: {
        'max-lines': [
          'error',
          {
            max: 400,
            skipBlankLines: true,
            skipComments: true,
          },
        ],
      },
    },
  ],
};
```

If a component approaches 400 lines, split it into:
- Child presentation components (e.g., `BookingSummaryCard.tsx`)
- Custom business/state hooks (e.g., `useBookingDraft.ts`)
- Validation schema files (e.g., `booking.schema.ts`)

## 2. Feature Boundary Enforcement

Features must remain self-contained. A feature must never import the private internals of another feature.

Use `eslint-plugin-import` with `no-restricted-paths`:

```javascript
module.exports = {
  plugins: ['import'],
  rules: {
    'import/no-restricted-paths': [
      'error',
      {
        zones: [
          // Prevent features from reaching into other features
          {
            target: './src/features/booking',
            from: './src/features',
            except: ['./booking'],
            message: 'Direct cross-feature imports are prohibited. Expose public contracts through platform/ or shared/.',
          },
          {
            target: './src/features/loyalty',
            from: './src/features',
            except: ['./loyalty'],
            message: 'Direct cross-feature imports are prohibited.',
          },
          // Shared cannot depend on features or app
          {
            target: './src/shared',
            from: './src/features',
            message: 'Shared modules must not depend on feature modules.',
          },
        ],
      },
    ],
  },
};
```

## 3. Strict TypeScript and Code Hygiene Rules

```javascript
module.exports = {
  rules: {
    // Prohibit unsafe 'any'
    '@typescript-eslint/no-explicit-any': 'error',

    // Prohibit ignoring type errors without explicit approved rationale
    '@typescript-eslint/ban-ts-comment': [
      'error',
      {
        'ts-ignore': true,
        'ts-nocheck': true,
      },
    ],

    // Enforce explicit return types on API calls and hooks
    '@typescript-eslint/explicit-function-return-type': [
      'warn',
      {
        allowExpressions: true,
        allowTypedFunctionExpressions: true,
      },
    ],

    // Prohibit mutating props or state
    'no-param-reassign': 'error',

    // Enforce React Hook rules
    'react-hooks/rules-of-hooks': 'error',
    'react-hooks/exhaustive-deps': 'error',
  },
};
```

## 4. Accessibility and Semantic Element Enforcement

Enforce semantic HTML using `eslint-plugin-jsx-a11y`:

```javascript
module.exports = {
  plugins: ['jsx-a11y'],
  extends: ['plugin:jsx-a11y/recommended'],
  rules: {
    // Prohibit fake buttons made from div/span
    'jsx-a11y/no-static-element-interactions': 'error',
    'jsx-a11y/click-events-have-key-events': 'error',

    // Ensure all images have meaningful alt text
    'jsx-a11y/alt-text': 'error',

    // Ensure interactive elements have accessible names
    'jsx-a11y/aria-role': 'error',
  },
};
```

## 5. CI Execution Gate

1. `npm run lint` and `npm run type-check` run in CI for every pull request.
2. Any warning or error fails the pipeline.
3. Pre-commit hooks (`husky` + `lint-staged`) run these checks locally before code can be committed.
