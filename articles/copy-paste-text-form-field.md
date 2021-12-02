---
title: "コピペで使えるTextFormField"
emoji: "✍️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [flutter]
published: true
---

入力フォームの作り方を毎度忘れてしまうので、思いつく限りのデザイン集を作って、コピペで楽しようよという記事です。師走なので。

# デフォルト

![](/images/text_form_field/1.png =400x)

```dart
TextFormField(
  decoration: const InputDecoration(hintText: 'デフォルト'),
)
```

---

# 背景色あり、フレームあり

![](/images/text_form_field/2.png =400x)

```dart
TextFormField(
  decoration: InputDecoration(
    hintText: '背景色あり, フレームあり',
    hintStyle: const TextStyle(fontSize: 12, color: Colors.green),
    fillColor: Colors.green[100],
    filled: true,
    focusedBorder: OutlineInputBorder(
      borderRadius: BorderRadius.circular(16),
      borderSide: const BorderSide(
        color: Colors.green,
        width: 2.0,
      ),
    ),
    enabledBorder: OutlineInputBorder(
      borderRadius: BorderRadius.circular(16),
      borderSide: BorderSide(
        color: Colors.green[100]!,
        width: 1.0,
      ),
    ),
  ),
),
```

---

# フレームあり、ラベルあり

![](/images/text_form_field/3.png =400x)

```dart
TextFormField(
  decoration: InputDecoration(
    focusedBorder: OutlineInputBorder(
      borderRadius: BorderRadius.circular(16),
      borderSide: const BorderSide(
        color: Colors.green,
        width: 2.0,
      ),
    ),
    labelStyle: TextStyle(
      fontSize: 12,
      color: Colors.green[300],
    ),
    labelText: 'フレームあり、ラベルあり',
    floatingLabelStyle: const TextStyle(fontSize: 12),
    enabledBorder: OutlineInputBorder(
      borderRadius: BorderRadius.circular(16),
      borderSide: BorderSide(
        color: Colors.green[100]!,
        width: 1.0,
      ),
    ),
  ),
),
```

---

# アイコンあり、フレームあり

![](/images/text_form_field/4.png =400x)

```dart
TextFormField(
  decoration: const InputDecoration(
    hintText: 'アイコンあり、フレームあり',
    prefixIcon: Icon(
      Icons.search,
    ),
    border: OutlineInputBorder(),
  ),
),
```

---

# アイコンあり、フレームなし

![](/images/text_form_field/5.png =400x)

```dart
TextFormField(
  decoration: InputDecoration(
    hintText: 'アイコンあり、フレームなし',
    fillColor: Colors.green[100],
    filled: true,
    isDense: true,
    prefixIcon: const Icon(Icons.search),
    border: OutlineInputBorder(
      borderRadius: BorderRadius.circular(32),
      borderSide: BorderSide.none,
    ),
  ),
),
```

---

# フレームなし、パディングなし

![](/images/text_form_field/6.png =400x)

```dart
TextFormField(
  decoration: InputDecoration.collapsed(
    hintText: 'フレームなし、パディングなし',
    fillColor: Colors.green[100],
    filled: true,
  ),
),
```

---

# 複数行入力、フレームなし、パディングなし

![](/images/text_form_field/7.png =400x)

```dart
TextFormField(
  maxLines: 6,
  minLines: 6,
  keyboardType: TextInputType.multiline,
  decoration: InputDecoration.collapsed(
    hintText: '複数行入力\n1\n2\n3\n4\n5',
    fillColor: Colors.green[100],
    filled: true,
  ),
),
```

---

# 複数行入力、角フレームあり、ラベルあり

![](/images/text_form_field/8.png =400x)

```dart
TextFormField(
  maxLines: 6,
  minLines: 6,
  keyboardType: TextInputType.multiline,
  decoration: InputDecoration(
    labelText: '複数行、角フレームあり、ラベルあり',
    enabledBorder: OutlineInputBorder(
      borderRadius: BorderRadius.circular(0),
      borderSide: const BorderSide(
        width: 2,
        color: Colors.green,
      ),
    ),
    focusedBorder: OutlineInputBorder(
      borderRadius: BorderRadius.circular(0),
      borderSide: const BorderSide(
        width: 2,
        color: Colors.green,
      ),
    ),
  ),
),
```

# パラメータ解説

coming soon...
