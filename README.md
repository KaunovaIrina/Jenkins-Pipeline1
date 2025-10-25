# Jenkins Pipeline — README

**Цель проекта:** автоматизация процесса сборки, тестирования и деплоя приложения с помощью **Jenkins Pipeline**.  
Этот репозиторий содержит пример `Jenkinsfile`, описывающий stages, steps и параметры для CI/CD. ⚙️

> Jenkins Pipeline — это скриптовая DSL-конфигурация, описывающая весь жизненный цикл сборки и доставки кода: от клонирования репозитория до деплоя в прод.

---

## Основные стадии пайплайна

| Стадия | Назначение | Основные действия |
|---|---|---|
| **Checkout** | Получение исходного кода | Клонирование Git-репозитория |
| **Build** | Сборка приложения | Компиляция, установка зависимостей |
| **Test** | Тестирование | Запуск unit/integration тестов |
| **Docker Build** | Подготовка образа | Сборка и тегирование Docker-образа |
| **Deploy** | Деплой | Развертывание на staging/production |
| **Post Actions** | Очистка и уведомления | Slack-уведомление, отчёты, cleanup |

---

## Пример `Jenkinsfile`

```groovy
pipeline {
    agent any

    parameters {
        string(name: 'BRANCH_NAME', defaultValue: 'main', description: 'Git ветка для сборки')
        choice(name: 'DEPLOY_ENV', choices: ['staging', 'production'], description: 'Окружение деплоя')
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: "${params.BRANCH_NAME}", url: 'https://github.com/example/app-repo.git'
            }
        }

        stage('Build') {
            steps {
                sh 'npm install'
                sh 'npm run build'
            }
        }

        stage('Test') {
            steps {
                sh 'npm test'
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t myapp:${env.BUILD_NUMBER} .'
            }
        }

        stage('Deploy') {
            when {
                expression { params.DEPLOY_ENV != '' }
            }
            steps {
                sh 'echo "Deploying to ${params.DEPLOY_ENV}..."'
                sh './scripts/deploy.sh ${params.DEPLOY_ENV}'
            }
        }
    }

    post {
        always {
            echo 'Pipeline завершён ✅'
            cleanWs()
        }
    }
}
```

> 💡 Примечание: замените `https://github.com/example/app-repo.git` на URL вашего проекта.

---

## Инструкции по запуску

### 🧱 Предварительные требования
- Установленный Jenkins (v2.300+)
- Плагины: *Pipeline*, *Git*, *Docker Pipeline*
- Доступ к Docker (на агенте)

### ▶️ Запуск пайплайна
1. Открой Jenkins → **New Item** → выбери **Pipeline**
2. Укажи название проекта (например, `app-ci-pipeline`)
3. В разделе **Pipeline → Definition** выбери **Pipeline script from SCM**
4. Тип SCM: **Git**
5. URL: `https://github.com/<username>/jenkins-pipeline-project.git`
6. Script Path: `pipeline/Jenkinsfile`

После сохранения нажми **Build Now** — Jenkins выполнит все стадии по описанию в Jenkinsfile 🚀

---

## Настройка параметров

- **BRANCH_NAME** — ветка для сборки (по умолчанию `main`)
- **DEPLOY_ENV** — окружение для деплоя (`staging` / `production`)

Пример запуска с параметрами:
```bash
curl -X POST JENKINS_URL/job/app-ci-pipeline/buildWithParameters   --user user:token   --data BRANCH_NAME=develop   --data DEPLOY_ENV=staging
```

---

## Полезные ссылки
- [Документация Jenkins Pipeline](https://www.jenkins.io/doc/book/pipeline/)
- [Declarative Pipeline Syntax](https://www.jenkins.io/doc/book/pipeline/syntax/)
- [Best Practices for Jenkinsfiles](https://www.jenkins.io/doc/book/pipeline/best-practices/)

---

📘 *Проект создан для демонстрации принципов CI/CD и может быть адаптирован под любую технологию (Node.js, Java, Python, Go и т.д.).*
