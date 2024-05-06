# Task-30
test


# myapp/models.py
from django.db import models

class MyModel(models.Model):
    name = models.CharField(max_length=100)
    description = models.TextField()

    def __str__(self):
        return self.name

# myapp/tests.py
from django.test import TestCase, Client
from .models import MyModel
from django.urls import reverse
from .forms import MyModelForm

class MyModelTestCase(TestCase):
    def setUp(self):
        self.obj = MyModel.objects.create(name="Test", description="This is a test.")

    def test_model_can_be_saved(self):
        self.assertEqual(MyModel.objects.count(), 1)

    def test_model_can_be_retrieved(self):
        obj = MyModel.objects.get(name="Test")
        self.assertEqual(obj.description, "This is a test.")

    def test_model_can_be_updated(self):
        self.obj.description = "Updated description"
        self.obj.save()
        obj = MyModel.objects.get(name="Test")
        self.assertEqual(obj.description, "Updated description")

    def test_model_can_be_deleted(self):
        self.obj.delete()
        self.assertEqual(MyModel.objects.count(), 0)

class MyViewTestCase(TestCase):
    def setUp(self):
        self.obj = MyModel.objects.create(name="Test", description="This is a test.")
        self.client = Client()

    def test_list_view(self):
        response = self.client.get(reverse('my_model_list'))
        self.assertEqual(response.status_code, 200)
        self.assertQuerysetEqual(response.context['objects'], [repr(self.obj)])

class MyFormTestCase(TestCase):
    def test_valid_form(self):
        form = MyModelForm(data={'name': 'Test', 'description': 'This is a test.'})
        self.assertTrue(form.is_valid())

    def test_invalid_form(self):
        form = MyModelForm(data={})
        self.assertFalse(form.is_valid())
        self.assertEqual(len(form.errors), 2)  # Both fields are required


# myapp/views.py
from django.shortcuts import render, redirect
from .models import MyModel

def my_model_list(request):
    objects = MyModel.objects.all()
    return render(request, 'myapp/my_model_list.html', {'objects': objects})

# myapp/forms.py
from django import forms
from .models import MyModel

class MyModelForm(forms.ModelForm):
    class Meta:
        model = MyModel
        fields = ['name', 'description']


