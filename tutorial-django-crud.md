# Tutorial Django - Primeira Aula: CRUD de Usuários

## Objetivo
Criar uma aplicação Django simples com CRUD (Create, Read, Update, Delete) para gerenciar usuários com email e senha.

---

## Pré-requisitos
- Python 3.8 ou superior instalado
- pip (gerenciador de pacotes Python)

---

## Passo 1: Configurar o Ambiente

### 1.1 Criar e ativar ambiente virtual
```bash
# Criar ambiente virtual
python3 -m venv venv

# Ativar no Linux/Mac
source venv/bin/activate

# Ativar no Windows
venv\Scripts\activate
```

### 1.2 Instalar Django
```bash
pip install django
```

---

## Passo 2: Criar o Projeto Django

```bash
# Criar projeto
django-admin startproject meu_projeto

# Entrar na pasta do projeto
cd meu_projeto

# Criar app de usuários
python manage.py startapp usuarios
```

---

## Passo 3: Configurar o App

### 3.1 Registrar o app no projeto
Edite o arquivo `meu_projeto/settings.py`:

```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'usuarios',  # Adicionar esta linha
]
```

---

## Passo 4: Criar o Model

Edite o arquivo `usuarios/models.py`:

```python
from django.db import models

class Usuario(models.Model):
    email = models.EmailField(unique=True, max_length=255)
    senha = models.CharField(max_length=128)
    
    def __str__(self):
        return self.email
    
    class Meta:
        verbose_name = 'Usuário'
        verbose_name_plural = 'Usuários'
```

---

## Passo 5: Criar e Aplicar Migrations

```bash
# Criar migrations
python manage.py makemigrations

# Aplicar migrations
python manage.py migrate
```

---

## Passo 6: Criar as Views

Edite o arquivo `usuarios/views.py`:

```python
from django.shortcuts import render, redirect, get_object_or_404
from .models import Usuario

# Listar todos os usuários (READ)
def listar_usuarios(request):
    usuarios = Usuario.objects.all()
    return render(request, 'usuarios/listar.html', {'usuarios': usuarios})

# Criar novo usuário (CREATE)
def criar_usuario(request):
    if request.method == 'POST':
        email = request.POST.get('email')
        senha = request.POST.get('senha')
        Usuario.objects.create(email=email, senha=senha)
        return redirect('listar_usuarios')
    return render(request, 'usuarios/criar.html')

# Editar usuário (UPDATE)
def editar_usuario(request, id):
    usuario = get_object_or_404(Usuario, id=id)
    if request.method == 'POST':
        usuario.email = request.POST.get('email')
        usuario.senha = request.POST.get('senha')
        usuario.save()
        return redirect('listar_usuarios')
    return render(request, 'usuarios/editar.html', {'usuario': usuario})

# Deletar usuário (DELETE)
def deletar_usuario(request, id):
    usuario = get_object_or_404(Usuario, id=id)
    if request.method == 'POST':
        usuario.delete()
        return redirect('listar_usuarios')
    return render(request, 'usuarios/deletar.html', {'usuario': usuario})
```

---

## Passo 7: Configurar URLs

### 7.1 Criar arquivo `usuarios/urls.py`:

```python
from django.urls import path
from . import views

urlpatterns = [
    path('', views.listar_usuarios, name='listar_usuarios'),
    path('criar/', views.criar_usuario, name='criar_usuario'),
    path('editar/<int:id>/', views.editar_usuario, name='editar_usuario'),
    path('deletar/<int:id>/', views.deletar_usuario, name='deletar_usuario'),
]
```

### 7.2 Incluir URLs do app no projeto
Edite o arquivo `meu_projeto/urls.py`:

```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('usuarios/', include('usuarios.urls')),
]
```

---

## Passo 8: Criar Templates

### 8.1 Criar estrutura de pastas
```bash
mkdir -p usuarios/templates/usuarios
```

### 8.2 Criar template base `usuarios/templates/usuarios/base.html`:

```html
<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>CRUD Usuários</title>
    <style>
        body { font-family: Arial, sans-serif; margin: 20px; }
        table { border-collapse: collapse; width: 100%; }
        th, td { border: 1px solid #ddd; padding: 8px; text-align: left; }
        th { background-color: #4CAF50; color: white; }
        a { color: #4CAF50; text-decoration: none; margin-right: 10px; }
        a:hover { text-decoration: underline; }
        button { background-color: #4CAF50; color: white; padding: 10px 20px; 
                 border: none; cursor: pointer; }
        button:hover { background-color: #45a049; }
        .delete-btn { background-color: #f44336; }
        .delete-btn:hover { background-color: #da190b; }
    </style>
</head>
<body>
    <h1>Gerenciamento de Usuários</h1>
    {% block content %}{% endblock %}
</body>
</html>
```

### 8.3 Criar `usuarios/templates/usuarios/listar.html`:

```html
{% extends 'usuarios/base.html' %}

{% block content %}
<a href="{% url 'criar_usuario' %}"><button>Criar Novo Usuário</button></a>
<br><br>

<table>
    <thead>
        <tr>
            <th>ID</th>
            <th>Email</th>
            <th>Ações</th>
        </tr>
    </thead>
    <tbody>
        {% for usuario in usuarios %}
        <tr>
            <td>{{ usuario.id }}</td>
            <td>{{ usuario.email }}</td>
            <td>
                <a href="{% url 'editar_usuario' usuario.id %}">Editar</a>
                <a href="{% url 'deletar_usuario' usuario.id %}">Deletar</a>
            </td>
        </tr>
        {% empty %}
        <tr>
            <td colspan="3">Nenhum usuário cadastrado.</td>
        </tr>
        {% endfor %}
    </tbody>
</table>
{% endblock %}
```

### 8.4 Criar `usuarios/templates/usuarios/criar.html`:

```html
{% extends 'usuarios/base.html' %}

{% block content %}
<h2>Criar Novo Usuário</h2>

<form method="POST">
    {% csrf_token %}
    <label>Email:</label><br>
    <input type="email" name="email" required><br><br>
    
    <label>Senha:</label><br>
    <input type="password" name="senha" required><br><br>
    
    <button type="submit">Criar</button>
    <a href="{% url 'listar_usuarios' %}"><button type="button">Cancelar</button></a>
</form>
{% endblock %}
```

### 8.5 Criar `usuarios/templates/usuarios/editar.html`:

```html
{% extends 'usuarios/base.html' %}

{% block content %}
<h2>Editar Usuário</h2>

<form method="POST">
    {% csrf_token %}
    <label>Email:</label><br>
    <input type="email" name="email" value="{{ usuario.email }}" required><br><br>
    
    <label>Senha:</label><br>
    <input type="password" name="senha" value="{{ usuario.senha }}" required><br><br>
    
    <button type="submit">Salvar</button>
    <a href="{% url 'listar_usuarios' %}"><button type="button">Cancelar</button></a>
</form>
{% endblock %}
```

### 8.6 Criar `usuarios/templates/usuarios/deletar.html`:

```html
{% extends 'usuarios/base.html' %}

{% block content %}
<h2>Confirmar Exclusão</h2>

<p>Tem certeza que deseja deletar o usuário <strong>{{ usuario.email }}</strong>?</p>

<form method="POST">
    {% csrf_token %}
    <button type="submit" class="delete-btn">Sim, Deletar</button>
    <a href="{% url 'listar_usuarios' %}"><button type="button">Cancelar</button></a>
</form>
{% endblock %}
```

---

## Passo 9: Executar o Servidor

```bash
python manage.py runserver
```

Acesse no navegador: `http://127.0.0.1:8000/usuarios/`

---

## Passo 10: (Opcional) Registrar no Admin do Django

Edite o arquivo `usuarios/admin.py`:

```python
from django.contrib import admin
from .models import Usuario

@admin.register(Usuario)
class UsuarioAdmin(admin.ModelAdmin):
    list_display = ('id', 'email')
    search_fields = ('email',)
```

### Criar superusuário:
```bash
python manage.py createsuperuser
```

Acesse o admin em: `http://127.0.0.1:8000/admin/`

---

## Estrutura Final do Projeto

```
meu_projeto/
├── meu_projeto/
│   ├── __init__.py
│   ├── settings.py
│   ├── urls.py
│   └── wsgi.py
├── usuarios/
│   ├── migrations/
│   ├── templates/
│   │   └── usuarios/
│   │       ├── base.html
│   │       ├── listar.html
│   │       ├── criar.html
│   │       ├── editar.html
│   │       └── deletar.html
│   ├── __init__.py
│   ├── admin.py
│   ├── apps.py
│   ├── models.py
│   ├── urls.py
│   └── views.py
└── manage.py
```

---

## Conceitos Importantes Abordados

1. **Model**: Representa a estrutura de dados (tabela do banco)
2. **View**: Lógica de negócio e processamento
3. **Template**: Interface visual (HTML)
4. **URLs**: Roteamento das requisições
5. **CRUD**: Create, Read, Update, Delete
6. **Migrations**: Versionamento do banco de dados

---

## Próximos Passos

- Adicionar validações de formulário com Django Forms
- Implementar autenticação de usuários
- Adicionar CSS com Bootstrap
- Usar hash para senhas (bcrypt/Django Auth)
- Implementar mensagens de sucesso/erro

---

## Observação Importante

⚠️ **Segurança**: Este exemplo é didático. Em produção, NUNCA armazene senhas em texto puro! Use o sistema de autenticação do Django (`django.contrib.auth`) que já inclui hash de senhas.
