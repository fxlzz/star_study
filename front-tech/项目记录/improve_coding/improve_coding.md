> 从这个名字即可看出，这是用于提升coding能力的，为我也为开发者

## 需求分析

## 系统设计

## 初始化项目
- `npm init`
- `.gitignore`
```js
	node_modules
	package-lock.json
```
- `eslintrc`
```json
{
  "extends": ["plugin:vue/base", "plugin:vue/recommended"],
  "plugins": ["vue"],
  "env": {
    "browser": true,
    "node": true,
  },
  "parser": "vue-eslint-parser",
  "parserOptions": {
    "parser": "babel-eslint",
    "ecmaVersion": 2017,
    "sourceType": "module",
  },

  "rules": {
    "no-unused-vars": [2, { "args": "none" }],
    "strict": "off",
    "valid-isdoc": "off",
    "jsdoc/require-param-description": "off",
    "jsdoc/require-param-type": "off",
    "jsdoc/check-param-names": "off",
    "jsdoc/require-param": "off",
    "jsdoc/check-tag-names": "off",
    "linebreak-style": "off",
    "array-bracket-spacing": "off",
    "prefer-promise-reject-errors": "off",
    "comma-dangle": "off",
    "newline-per-chained-call": "off",
    "no-loop-func": "off",
    "no-empty": "off",
    "no-else-return": "off",
    "no-unneeded-ternary": "off",
    "no-eval": "off",
    "prefer-destructuring": "off",
    "no-param-reassign": "off",
    "max-len": "off",
    "no-restricted-syntax": "off",
    "no-plusplus": "off",
    "no-useless-escape": "off",
    "no-nested-ternary": "off",
    "radix": "off",
    "arrow-body-style": "off",
    "arrow-parens": "off",
    "vue/multi-word-component-names": "off",
    "vue/valid-v-for": "off",
    "vue/no-multiple-template-root": "off",
  },
  "globals": {
    "$": true,
    "axios": true,
    "Vue": true,
  },
}
```
- `eslintignore`
```js
node_modules/
public/
```
配置`script`: `"lint": "eslint --quiet --ext js,vue ."` 检查当前目录下的 .js .vue 文件

- `validate-commit-msg` & `ghooks`
	规范 git 提交提示词，并且将其他规范工具集成到提交之前（eslint..)
```json
// 在 package.json 文件中写入
"config": {
    "ghooks": {
      "commit-msg": "validate-commit-msg",
      "pre-commit": "npm run lint"
    }
  }
```