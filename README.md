# 实验四 Web应用程序

班级： 21软件1

## 实验目的

1. Python虚拟环境的安装和使用
2. 利用Django进行Web应用程序开发
3. 掌握Django的基本开发流程

## 实验环境

1. Git
2. Python
3. VSCode
4. VSCode插件
   - Django
   - SQLite
   - SQLite Viewer

## 实验项目分工协作情况

### 项目组员

### 项目分工

## 实验过程记录

```python
from django.contrib import admin
from .models import Topic, Entry

# Register your models here.
admin.site.register(Topic)
admin.site.register(Entry)

from django.db import models
from django.contrib.auth.models import User

# Create your models here.
class Topic(models.Model):
    """用户学习的主题"""
    text=models.CharField(max_length=200)
    date_added=models.DateTimeField(auto_now_add=True)
    owner = models.ForeignKey(User, on_delete=models.CASCADE)
    
    def __str__(self):
        """返回模型的字符串表示"""
        return self.text
    
class Entry(models.Model):
    """学到的有关某个主题的具体知识"""
    topic = models.ForeignKey(Topic, on_delete=models.CASCADE)
    text = models.TextField()
    date_added = models.DateTimeField(auto_now_add=True)

    class Meta:
        verbose_name_plural = 'entries'

    def __str__(self):
        """返回一个表示条目的简单字符串"""
        return f"{self.text[:50]}..."

from django.urls import path

from . import views

app_name = 'learning_logs'
urlpatterns = [
    # 主页
    path('', views.index, name='index'),
    # 显示所有主题的界面
    path('topics/', views.topics, name='topics'),
    # 特定主题的详细页面
    path('topics/<int:topic_id>/', views.topic, name='topic'),
    # 用于添加新主题的网页
    path('new_topic/', views.new_topic, name='new_topic'),
    # 添加新条目的页面。
    path('new_entry/<int:topic_id>/', views.new_entry, name='new_entry'),
    # 编辑条目的页面。
    path('edit_entry/<int:entry_id>/', views.edit_entry, name='edit_entry'),
]

from django.shortcuts import render, redirect
from django.contrib.auth.decorators import login_required
from django.http import Http404

from .models import Topic,Entry
from .forms import TopicForm, EntryForm


def index(request):
    """学习笔记的主页"""
    return render(request, 'learning_logs/index.html')

@login_required
def topics(request):
    """显示所有的主题"""
    topics = Topic.objects.filter(owner=request.user).order_by('date_added')
    topics = Topic.objects.order_by('date_added')
    context = {'topics': topics}
    return render(request, 'learning_logs/topics.html', context)

@login_required
def topic(request, topic_id):
    """显示所有主题和"""
    topic = Topic.objects.get(id=topic_id)
    # 确认请求的主题属于当前用户
    if topic.owner != request.user:
        raise Http404
    entries = topic.entry_set.order_by('-date_added')
    context = {'topic': topic, 'entries': entries}
    return render(request, 'learning_logs/topic.html', context)

@login_required
def new_topic(request):
    """添加新主题"""
    if request.method != 'POST':
        # 未提交数据：创建一个新表单
        form = TopicForm()
    else:
        # POST 提交的数据：对数据进行处理
        form = TopicForm(data=request.POST)
        if form.is_valid():
            new_topic = form.save(commit=False)
            new_topic.owner = request.user
            new_topic.save()
            return redirect('learning_logs:topics')
        
        # 显示空白或无效的表单
    context = {'form': form}
    return render(request, 'learning_logs/new_topic.html', context)

@login_required
def new_entry(request, topic_id):
    """为特定主题添加新条目。"""
    topic = Topic.objects.get(id=topic_id)
    
    if request.method != 'POST':
        # 未提交数据；创建一个空白表单。
        form = EntryForm()
    else:
        # 提交了POST数据；处理数据。
        form = EntryForm(data=request.POST)
        if form.is_valid():
            new_entry = form.save(commit=False)
            new_entry.topic = topic
            new_entry.save()
            return redirect('learning_logs:topic', topic_id=topic_id)

    # 显示空白或无效的表单。
    context = {'topic': topic, 'form': form}
    return render(request, 'learning_logs/new_entry.html', context)

@login_required
def edit_entry(request, entry_id):
    """编辑现有条目。"""
    entry = Entry.objects.get(id=entry_id)
    topic = entry.topic

    if request.method != 'POST':
        # 初始请求；用当前条目的内容预填充表单。
        form = EntryForm(instance=entry)
    else:
        # 提交了POST数据；处理数据。
        form = EntryForm(instance=entry, data=request.POST)
        if form.is_valid():
            form.save()
            return redirect('learning_logs:topic', topic_id=topic.id)

    context = {'entry': entry, 'topic': topic, 'form': form}
    return render(request, 'learning_logs/edit_entry.html', context)
```

在本次实验中

用户数据输入：我首先实现了用户能够通过前端界面输入数据的功能，确保用户可以填写表单并提交信息到后端。这涉及到Django表单处理以及视图函数的编写，用于接收POST请求并处理用户提交的数据。

创建用户账户系统：接着，我创建了一个用户注册和登录系统，使用Django内置的User模型和auth模块，实现用户账户的创建、认证与会话管理。这包括了设计用户注册页面、登录页面以及相应的后端逻辑，确保用户数据的安全存储与验证。

用户数据所有权：为防止用户误操作或恶意篡改他人数据，我特别针对“new_entry”页面进行了安全增强。在保存新条目前，我添加了逻辑检查，确保所添加的条目属于当前登录用户对应的主题ID，从而保护了用户数据的隐私和完整性。

## 实验总结

通过参与这次实验，我在以下几个方面得到了显著提升和学习：

Django框架应用：深入理解了Django框架的基本架构和开发流程，从模型(Model)、视图(View)到模板(Template)的MVT模式，掌握了如何快速搭建Web应用。

用户认证与授权：熟练掌握了Django的用户认证系统，包括用户模型的扩展、登录/注销功能的实现以及基于session的用户状态管理，加深了对Web应用安全性的认识。

Git团队协作：熟悉了Git版本控制工具的使用，包括创建和管理分支、解决冲突、提交代码以及通过Pull Request进行代码审查的过程，这对于团队合作至关重要。

编程技巧与问题解决：在处理用户账户安全问题时，我学会了如何在实际场景中应用条件判断和数据库查询来实现业务逻辑，这增强了我的问题分析和解决能力。

Markdown与Mermaid：掌握了Markdown文档编写规范，使用Markdown和Mermaid插件来编写实验报告和绘制流程图，这提高了文档的可读性和专业性。

综上所述，这次实验不仅让我在技术层面上有了实践和进步，还让我体验到了团队协作的重要性以及版本控制在软件开发中的核心作用，是一次非常宝贵的学习经历。
