# Pneumonia Detection AI

## Spis treści
- [Opis projektu](#opis-projektu)
- [Struktura projektu](#struktura-projektu)
- [Wymagania](#wymagania)
- [Instalacja](#instalacja)
- [Przygotowanie danych](#przygotowanie-danych)
- [Trenowanie modelu](#trenowanie-modelu)
- [Ocena modelu](#ocena-modelu)
- [Zapisanie modelu](#zapisanie-modelu)
- [Wykonanie predykcji](#wykonanie-predykcji)
- [Wizualizacja wyników](#wizualizacja-wyników)
- [Autorzy](#autorzy)

## Opis projektu
Projekt "Pneumonia Detection AI" ma na celu stworzenie modelu sieci konwolucyjnej (CNN) do wykrywania zapalenia płuc na podstawie zdjęć rentgenowskich klatki piersiowej. Model jest trenowany, walidowany i testowany na zbiorze danych zawierającym obrazy zdrowych płuc oraz płuc z zapaleniem.

## Struktura projektu
```
pneumonia-detection-ai/
│
├── chest_xray/
│   ├── test/
│   │   ├── NORMAL/
│   │   └── PNEUMONIA/
│   ├── train/
│   │   ├── NORMAL/
│   │   └── PNEUMONIA/
│   └── val/
│       ├── NORMAL/
│       └── PNEUMONIA/
│
├── photos/
│   ├── chory.jpeg
│   ├── stopy.jpg
│   └── zdrowy.jpeg
│
├── .gitattributes
├── notebook.ipynb
├── requirements.txt
├── pneumonia_detection_model.h5
└── README.md
```

## Wymagania
- Python 3.7+
- TensorFlow 2.0+
- NumPy
- Matplotlib
- Seaborn
- Scikit-learn

## Instalacja
1. Sklonuj repozytorium:
   ```sh
   git clone https://github.com/LiCHUTKO/pneumonia-detection-ai.git
   cd pneumonia-detection-ai
   ```

2. Zainstaluj wymagane biblioteki:
   ```sh
   pip install -r requirements.txt
   ```

## Przygotowanie danych
Dane są zorganizowane w trzech katalogach: `train`, `val` i `test`, z podkatalogami `NORMAL` i `PNEUMONIA` w każdym z nich. Każdy podkatalog zawiera obrazy rentgenowskie klatki piersiowej.

## Trenowanie modelu
1. Otwórz plik notebook.ipynb w Jupyter Notebook lub Jupyter Lab.
2. Wykonaj wszystkie komórki w notebooku, aby załadować dane, przygotować je, zbudować model, przeprowadzić trening i ocenić model.

### Kod do trenowania modelu:
```python
# Budowanie modelu sieci konwolucyjnej
model = Sequential([
    Conv2D(32, (3, 3), activation='relu', input_shape=(150, 150, 3)),
    MaxPooling2D((2, 2)),
    Dropout(0.5),
    Conv2D(64, (3, 3), activation='relu'),
    MaxPooling2D((2, 2)),
    Dropout(0.5),
    Conv2D(128, (3, 3), activation='relu'),
    MaxPooling2D((2, 2)),
    Dropout(0.5),
    Flatten(),
    Dense(512, activation='relu'),
    Dropout(0.5),
    Dense(1, activation='sigmoid')
])

model.compile(optimizer='adam', loss='binary_crossentropy', metrics=['accuracy'])

# Trenowanie modelu
early_stopping = EarlyStopping(monitor='val_loss', patience=5, restore_best_weights=True)

history = model.fit(
    train_generator,
    steps_per_epoch=train_generator.samples // train_generator.batch_size,
    validation_data=val_generator,
    validation_steps=val_generator.samples // val_generator.batch_size,
    epochs=20,
    callbacks=[early_stopping]
)
```

## Ocena modelu
Model jest oceniany na zbiorze testowym za pomocą różnych metryk, takich jak dokładność, precyzja, czułość i F1-score.

### Kod do oceny modelu:
```python
test_loss, test_acc = model.evaluate(test_generator, steps=test_generator.samples // test_generator.batch_size)
print(f"Test accuracy: {test_acc:.2f}")

y_pred = model.predict(test_generator)
y_pred_classes = np.where(y_pred > 0.5, 1, 0).flatten()
y_true = test_generator.classes

accuracy = accuracy_score(y_true, y_pred_classes)
precision = precision_score(y_true, y_pred_classes)
recall = recall_score(y_true, y_pred_classes)
f1 = f1_score(y_true, y_pred_classes)

print(f"Dokładność (Accuracy): {accuracy:.2f}")
print(f"Precyzja (Precision): {precision:.2f}")
print(f"Czułość (Recall): {recall:.2f}")
print(f"F1-score: {f1:.2f}")
```

## Zapisanie modelu
Wytrenowany model jest zapisywany do pliku pneumonia_detection_model.h5, aby można go było później załadować i używać do predykcji.

### Kod do zapisania modelu:
```python
model.save('pneumonia_detection_model.h5')
```

## Wykonanie predykcji
Model może wykonywać predykcje na nowych obrazach załadowanych przez użytkownika.

### Kod do wykonania predykcji:
```python
from tensorflow.keras.preprocessing import image

def predict_image(img_path, model, class_names):
    img = image.load_img(img_path, target_size=(150, 150))
    img_array = image.img_to_array(img)
    img_array = np.expand_dims(img_array, axis=0)
    img_array /= 255.0

    prediction = model.predict(img_array)
    predicted_class = class_names[int(prediction > 0.5)]
    return predicted_class

# Przykład użycia funkcji predict_image
img_path = 'photos/zdrowy.jpeg'  # Zmień tę ścieżkę na ścieżkę do swojego obrazu
predicted_class = predict_image(img_path, model, class_names)
print(f"Predykcja: {predicted_class}")
```

## Wizualizacja wyników
Wyniki treningu i oceny modelu są wizualizowane za pomocą wykresów.

### Kod do wizualizacji wyników:
```python
# Wizualizacja wyników trenowania
acc = history.history['accuracy']
val_acc = history.history['val_accuracy']
loss = history.history['loss']
val_loss = history.history['val_loss']

epochs_range = range(len(acc))

plt.figure(figsize=(12, 8))
plt.subplot(1, 2, 1)
plt.plot(epochs_range, acc, label='Training Accuracy')
plt.plot(epochs_range, val_acc, label='Validation Accuracy')
plt.legend(loc='lower right')
plt.title('Training and Validation Accuracy')

plt.subplot(1, 2, 2)
plt.plot(epochs_range, loss, label='Training Loss')
plt.plot(epochs_range, val_loss, label='Validation Loss')
plt.legend(loc='upper right')
plt.title('Training and Validation Loss')
plt.show()

# Wizualizacja macierzy konfuzji
conf_matrix = confusion_matrix(y_true, y_pred_classes)
plt.figure(figsize=(10, 8))
sns.heatmap(conf_matrix, annot=True, fmt='d', cmap='Blues', xticklabels=['Zdrowy', 'Zapalenie płuc'], yticklabels=['Zdrowy', 'Zapalenie płuc'])
plt.xlabel('Przewidywane')
plt.ylabel('Rzeczywiste')
plt.title("Macierz konfuzji")
plt.show()
```

## Autorzy
- [LiCHUTKO](https://github.com/LiCHUTKO)