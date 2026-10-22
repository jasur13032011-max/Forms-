# Forms-
1. models.py — Modellar
Avval Tag modelini yaratamiz va larni Post modeliga ManyToManyField orqali bog'laymiz.

Python
from django.db import models


class Tag(models.Model):
  name = models.CharField(max_length=50, unique=True, verbose_name="Teg nomi")

  def __str__(self):
    return self.name


class Post(models.Model):
  title = models.CharField(max_length=200, verbose_name="Sarlavha")
  content = models.TextField(verbose_name="Matn")
  tags = models.ManyToManyField(Tag, blank=True, verbose_name="Teglar")
  created_at = models.DateTimeField(auto_now_add=True)

  def __str__(self):
    return self.title
2. admin.py — Admin Panel Sozlamalari
PostAdmin da list_display va search_fields maydonlarini sozlaymiz.

Python
from django.contrib import admin
from .models import Post, Tag

admin.site.register(Tag)


@admin.register(Post)
class PostAdmin(admin.ModelAdmin):
  list_display = ('title', 'created_at')
  search_fields = ('title', 'content')
3. forms.py — PostForm (ModelForm)
Post yaratish uchun ModelForm yaratamiz. Teglar maydoni ham kiritilishini ta'minlaymiz.

Python
from django import forms
from .models import Post


class PostForm(forms.ModelForm):

  class Meta:
    model = Post
    fields = ['title', 'content', 'tags']
    widgets = {
        # Teglarni qulay tanlash uchun Checkbox yoki SelectMultiple ishlatish mumkin
        'tags': forms.CheckboxSelectMultiple(),
    }
4. views.py — Postni saqlash jarayoni
form.save() orqali yangi postni yaratish va teglarni avtomatik ravishda saqlash:

Python
from django.shortcuts import redirect, render
from .forms import PostForm


def create_post(request):
  if request.method == 'POST':
    form = PostForm(request.POST)
    if form.is_valid():
      # form.save() avtomatik ravishda asosiy ma'lumotlarni ham, 
      # ManyToManyField (teglarni) ham bazaga to'g'ri saqlaydi.
      post = form.save()
      return redirect('success_url')  # Muvaffaqiyatli saqlangandan keyingi yo'nalish
  else:
    form = PostForm()

  return render(request, 'posts/create_post.html', {'form': form})
Eslatma: Agar formani commit=False qilib saqlagan taqdiringizda ham, teglarni saqlash uchun form.save_m2m() metodini chaqirish kerak bo'lar edi. Biroq oddiy form.save() chaqirilganda Django ManyToManyField ma'lumotlarini o'zi avtomatik ravishda to'liq saqlab beradi.
1. models.py
Python
from django.db import models


class Tag(models.Model):
  name = models.CharField(max_length=50, unique=True, verbose_name="Teg nomi")

  def __str__(self):
    return self.name


class Post(models.Model):
  title = models.CharField(max_length=200, verbose_name="Sarlavha")
  content = models.TextField(verbose_name="Matn")
  tags = models.ManyToManyField(Tag, blank=True, verbose_name="Teglar")
  created_at = models.DateTimeField(
      auto_now_add=True, verbose_name="Yaratilgan vaqti"
  )

  def __str__(self):
    return self.title
2. admin.py
Python
from django.contrib import admin
from .models import Post, Tag

# Tag modelini admin panelga ro'yxatdan o'tkazamiz
admin.site.register(Tag)


# Post modelini maxsus sozlamalar bilan ro'yxatdan o'tkazamiz
@admin.register(Post)
class PostAdmin(admin.ModelAdmin):
  list_display = ("title", "created_at")
  search_fields = ("title", "content")
3. forms.py
Python
from django import forms
from .models import Post


class PostForm(forms.ModelForm):

  class Meta:
    model = Post
    fields = ["title", "content", "tags"]
    widgets = {
        # Teglarni qulay tanlash uchun CheckboxSelectMultiple widget'idan foydalanamiz
        "tags": forms.CheckboxSelectMultiple(),
    }
4. views.py
Python
from django.shortcuts import redirect, render
from .forms import PostForm


def create_post_view(request):
  if request.method == "POST":
    form = PostForm(request.POST)
    if form.is_valid():
      # form.save() orqali post va uning ManyToManyField (teglari) avtomatik saqlanadi
      form.save()
      return redirect("post_list")  # Postlar ro'yxatiga yo'naltirish
  else:
    form = PostForm()

  context = {"form": form}
  return render(request, "posts/create_post.html", context)
5. Qo'shimcha: urls.py (App darajasida)
Python
from django.urls import path
from .views import create_post_view

urlpatterns = [
    path("create/", create_post_view, name="create_post"),
]
Shu fayllarni loyihangizga qo'shib, git orqali bosqichma-bosqich commit qilib yuborsangiz, o'qituvchi talab qilgan barcha shartlar to'liq bajarilgan bo'ladi!
