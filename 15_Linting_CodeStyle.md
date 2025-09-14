# 15. Linting & Code Style

## Tại sao cần Linting & Code Style?

### Lợi ích chính
- **Consistency**: Code nhất quán trong team
- **Quality**: Phát hiện bugs và code smells sớm
- **Maintainability**: Dễ đọc, bảo trì code
- **Collaboration**: Giảm conflicts khi merge code
- **Best Practices**: Enforce coding standards

### Vấn đề khi không có
- Code style khác nhau giữa developers
- Bugs tiềm ẩn không được phát hiện
- Code review mất thời gian cho formatting
- Khó maintain và debug

## ESLint - JavaScript/TypeScript Linter

### Cài đặt và khởi tạo
```bash
# Cài đặt ESLint
npm install eslint --save-dev

# Khởi tạo config
npx eslint --init

# Với TypeScript
npm install @typescript-eslint/parser @typescript-eslint/eslint-plugin --save-dev
```

### ESLint Configuration (.eslintrc.js)
```js
module.exports = {
  root: true,
  env: {
    node: true,
    es2021: true,
    jest: true
  },
  extends: [
    'eslint:recommended',
    '@typescript-eslint/recommended',
    'prettier' // Must be last
  ],
  parser: '@typescript-eslint/parser',
  parserOptions: {
    ecmaVersion: 2021,
    sourceType: 'module',
    project: './tsconfig.json'
  },
  plugins: ['@typescript-eslint'],
  rules: {
    // Error prevention
    'no-console': 'warn',
    'no-debugger': 'error',
    'no-unused-vars': 'off',
    '@typescript-eslint/no-unused-vars': 'error',
    
    // Code quality
    'prefer-const': 'error',
    'no-var': 'error',
    'object-shorthand': 'error',
    'prefer-template': 'error',
    
    // TypeScript specific
    '@typescript-eslint/no-explicit-any': 'warn',
    '@typescript-eslint/explicit-function-return-type': 'off',
    '@typescript-eslint/no-non-null-assertion': 'error',
    
    // Import/Export
    'import/order': ['error', {
      'groups': ['builtin', 'external', 'internal', 'parent', 'sibling'],
      'newlines-between': 'always'
    }]
  },
  overrides: [
    {
      files: ['*.test.ts', '*.test.js'],
      rules: {
        '@typescript-eslint/no-explicit-any': 'off'
      }
    }
  ]
};
```

### ESLint Scripts
```json
{
  "scripts": {
    "lint": "eslint . --ext .ts,.js",
    "lint:fix": "eslint . --ext .ts,.js --fix",
    "lint:quiet": "eslint . --ext .ts,.js --quiet"
  }
}
```

### Popular ESLint Configs
```bash
# Airbnb style guide
npm install eslint-config-airbnb-base --save-dev

# Standard JS
npm install eslint-config-standard --save-dev

# Google style guide
npm install eslint-config-google --save-dev

# React specific
npm install eslint-plugin-react eslint-plugin-react-hooks --save-dev
```

## Prettier - Code Formatter

### Cài đặt
```bash
# Prettier
npm install prettier --save-dev

# ESLint integration
npm install eslint-config-prettier eslint-plugin-prettier --save-dev
```

### Prettier Configuration (.prettierrc)
```json
{
  "semi": true,
  "trailingComma": "es5",
  "singleQuote": true,
  "printWidth": 80,
  "tabWidth": 2,
  "useTabs": false,
  "bracketSpacing": true,
  "arrowParens": "avoid",
  "endOfLine": "lf",
  "bracketSameLine": false,
  "quoteProps": "as-needed"
}
```

### Prettier Scripts
```json
{
  "scripts": {
    "format": "prettier --write .",
    "format:check": "prettier --check .",
    "format:staged": "prettier --write $(git diff --cached --name-only --diff-filter=ACMR | grep -E '\\.(ts|js|json|md)$')"
  }
}
```

### .prettierignore
```
node_modules/
dist/
build/
coverage/
.env
*.log
.git/
```

### ESLint + Prettier Integration
```js
// .eslintrc.js
module.exports = {
  extends: [
    'eslint:recommended',
    '@typescript-eslint/recommended',
    'prettier' // Disables ESLint rules that conflict with Prettier
  ],
  plugins: ['prettier'],
  rules: {
    'prettier/prettier': 'error' // Shows prettier errors as ESLint errors
  }
};
```

## Git Hooks với Husky

### Cài đặt Husky
```bash
npm install husky --save-dev
npm install lint-staged --save-dev

# Initialize husky
npx husky install

# Add to package.json
npm set-script prepare "husky install"
```

### Pre-commit Hook
```bash
# Add pre-commit hook
npx husky add .husky/pre-commit "npx lint-staged"
```

### lint-staged Configuration
```json
// package.json
{
  "lint-staged": {
    "*.{ts,js}": [
      "eslint --fix",
      "prettier --write"
    ],
    "*.{json,md}": [
      "prettier --write"
    ]
  }
}
```

### Pre-push Hook
```bash
npx husky add .husky/pre-push "npm run test && npm run lint"
```

### Commit Message Hook
```bash
npx husky add .husky/commit-msg "npx --no-install commitlint --edit $1"
```

## Commitlint - Commit Message Standards

### Cài đặt
```bash
npm install @commitlint/cli @commitlint/config-conventional --save-dev
```

### Configuration (commitlint.config.js)
```js
module.exports = {
  extends: ['@commitlint/config-conventional'],
  rules: {
    'type-enum': [
      2,
      'always',
      [
        'build',
        'chore', 
        'ci',
        'docs',
        'feat',
        'fix',
        'perf',
        'refactor',
        'revert',
        'style',
        'test'
      ]
    ],
    'subject-case': [2, 'always', 'sentence-case'],
    'subject-max-length': [2, 'always', 100],
    'body-max-line-length': [2, 'always', 100]
  }
};
```

### Conventional Commit Examples
```bash
# Good commits
feat: add user authentication
fix: resolve login redirect issue
docs: update API documentation
style: format code with prettier
refactor: extract user service
test: add unit tests for auth module
chore: update dependencies

# Bad commits
update stuff
fix bug
changes
wip
```

## EditorConfig - IDE Consistency

### .editorconfig
```ini
root = true

[*]
indent_style = space
indent_size = 2
end_of_line = lf
charset = utf-8
trim_trailing_whitespace = true
insert_final_newline = true

[*.md]
trim_trailing_whitespace = false

[*.{yml,yaml}]
indent_size = 2

[Makefile]
indent_style = tab
```

## Advanced Configuration

### Multiple Environments
```js
// .eslintrc.js
module.exports = {
  env: {
    node: true,
    browser: false,
    es2021: true
  },
  overrides: [
    {
      files: ['**/*.test.ts'],
      env: {
        jest: true
      }
    },
    {
      files: ['src/client/**/*.ts'],
      env: {
        browser: true,
        node: false
      }
    }
  ]
};
```

### Custom ESLint Rules
```js
// eslint-rules/no-console-log.js
module.exports = {
  create(context) {
    return {
      CallExpression(node) {
        if (
          node.callee.type === 'MemberExpression' &&
          node.callee.object.name === 'console' &&
          node.callee.property.name === 'log'
        ) {
          context.report({
            node,
            message: 'Use logger instead of console.log'
          });
        }
      }
    };
  }
};
```

### Workspace Configuration (Monorepo)
```js
// Root .eslintrc.js
module.exports = {
  root: true,
  extends: ['./eslint-base.js']
};

// packages/api/.eslintrc.js
module.exports = {
  extends: ['../../eslint-base.js'],
  env: {
    node: true,
    browser: false
  }
};

// packages/web/.eslintrc.js
module.exports = {
  extends: ['../../eslint-base.js'],
  env: {
    browser: true,
    node: false
  },
  plugins: ['react', 'react-hooks']
};
```

## CI/CD Integration

### GitHub Actions
```yaml
# .github/workflows/lint.yml
name: Lint & Format Check

on: [push, pull_request]

jobs:
  lint:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: '18'
        cache: 'npm'
    
    - name: Install dependencies
      run: npm ci
    
    - name: Run ESLint
      run: npm run lint
    
    - name: Check Prettier formatting
      run: npm run format:check
    
    - name: Run type check
      run: npm run type-check
```

### Pre-commit CI
```yaml
# .pre-commit-config.yaml
repos:
  - repo: local
    hooks:
      - id: eslint
        name: eslint
        entry: npx eslint --fix
        language: node
        files: \.(ts|js)$
        
      - id: prettier
        name: prettier
        entry: npx prettier --write
        language: node
        files: \.(ts|js|json|md)$
```

## VSCode Integration

### .vscode/settings.json
```json
{
  "editor.formatOnSave": true,
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": true,
    "source.organizeImports": true
  },
  "eslint.validate": [
    "javascript",
    "typescript"
  ],
  "typescript.preferences.organizeImportsIgnoreCase": "auto"
}
```

### Extensions
```json
// .vscode/extensions.json
{
  "recommendations": [
    "esbenp.prettier-vscode",
    "dbaeumer.vscode-eslint",
    "editorconfig.editorconfig",
    "streetsidesoftware.code-spell-checker"
  ]
}
```

## Best Practices

### 1. Team Consistency
- Share ESLint/Prettier configs across projects
- Use .editorconfig for IDE settings
- Document coding standards

### 2. Gradual Implementation
```bash
# Start with basic rules
npm install eslint --save-dev
npx eslint --init

# Add prettier later
npm install prettier eslint-config-prettier --save-dev

# Add git hooks last
npm install husky lint-staged --save-dev
```

### 3. Rule Configuration Strategy
```js
// Start strict, then relax
module.exports = {
  extends: ['eslint:recommended'],
  rules: {
    // Start with errors
    'no-console': 'error',
    'no-debugger': 'error',
    
    // Gradually add warnings
    'prefer-const': 'warn',
    
    // Turn off conflicting rules
    'semi': 'off'
  }
};
```

### 4. Performance Optimization
```js
// .eslintrc.js
module.exports = {
  // Cache results
  cache: true,
  cacheLocation: '.eslintcache',
  
  // Ignore large files
  ignorePatterns: [
    'dist/',
    'node_modules/',
    '*.min.js'
  ],
  
  // Use faster parser for JS files
  overrides: [
    {
      files: ['*.js'],
      parser: 'espree' // Instead of @typescript-eslint/parser
    }
  ]
};
```

## Troubleshooting

### Common Issues
```bash
# ESLint conflicts with Prettier
npm install eslint-config-prettier --save-dev

# TypeScript parsing errors
npm install @typescript-eslint/parser --save-dev

# Git hooks not working
npx husky install
chmod +x .husky/pre-commit

# Cache issues
npx eslint --cache-location .eslintcache
rm .eslintcache
```

### Debug Commands
```bash
# Check ESLint config
npx eslint --print-config file.ts

# Test single file
npx eslint src/index.ts --debug

# Prettier check
npx prettier --check src/index.ts --debug-check
```

## Câu hỏi phỏng vấn thường gặp

### 1. ESLint vs TSC (TypeScript Compiler)?
- **ESLint**: Code style, best practices, potential bugs
- **TSC**: Type checking, syntax errors, compilation

### 2. Khi nào dùng ESLint rules vs Prettier?
- **ESLint**: Logic, potential bugs, code quality
- **Prettier**: Formatting, spacing, brackets, quotes

### 3. Làm sao setup cho team lớn?
- Shared ESLint config package
- Centralized prettier config
- Git hooks enforcement
- CI/CD integration

### 4. Performance impact của linting?
- Use caching (`.eslintcache`)
- Run only on changed files (lint-staged)
- Parallel processing in CI
- Exclude unnecessary files

### 5. Conflict resolution ESLint vs Prettier?
- Use `eslint-config-prettier` to disable conflicting rules
- Run prettier after ESLint
- Use `prettier/prettier` ESLint rule

### 6. Custom rules khi nào cần?
- Company-specific patterns
- Framework-specific requirements  
- Security rules
- Performance optimizations
