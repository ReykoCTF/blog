---
title: "Auvergn'Hack 2025 : Sifaka"
date: 2025-04-06
draft: false
categories: ["Hardware"]
ctf: "Auvergn'Hack"
challenge: "Sifaka"
author: "Reyko"
---

# WU Sifaka - Hardware - Auvergn'Hack

Challenge : 
Sifaka

Enoncé : 
Crowned Sifaka has applied a Fourier transform to a wav audio file. The sample rate is 48000 and the transform has been stored as follows: {index}{real part}{imaginary part}. Find the message Sifaka wanted to transfer from the original wav file.

### Analyse du fichier

Le fichier `message.txt` contient des lignes telles que :

```
0	-6041920.0	0.0
1	1697623.641095433	-416966.6697899885
2	969244.4776290962	151676.52569088535
...
```
Chaque ligne représente le coefficient \( c[k] \) du spectre de Fourier :
\[
c[k] = \text{partie réelle} + i \times \text{partie imaginaire}
\]
L’index permet de reconstituer le vecteur complet des coefficients pour le signal transformé.

### 1. Reconstitution des coefficients de Fourier

- Lecture du fichier : Charger les données avec NumPy pour lire le fichier texte.
- Création des nombres complexes : Pour chaque ligne, combiner la partie réelle et la partie imaginaire pour former \( c[k] \).

### 2. Application de l’IFFT (Inverse Fast Fourier Transform)

- Passer du domaine fréquentiel (les coefficients de Fourier) au domaine temporel (signal audio).
- La fonction `np.fft.ifft` permet de récupérer le signal temporel.

### 3. Sauvegarde du signal audio

- Normalisation
- Création du fichier WAV : Utiliser la fonction `scipy.io.wavfile.write` pour écrire le signal dans un fichier audio en spécifiant le taux d’échantillonnage (48000 Hz).
- Durée approximative : La longueur totale du vecteur (par exemple 400800 coefficients) correspond à environ \( \frac{400800}{48000} \approx 8.35 \) secondes d’audio.

### 4. Analyse finale

Une fois le fichier WAV généré, il suffit de l’écouter pour découvrir le message transmis par Sifaka.

## Implem Python : 

```python
import numpy as np
from scipy.io.wavfile import write

data = np.loadtxt('message.txt', delimiter='\t')

fourier_coeffs = data[:, 1] + 1j * data[:, 2]

time_signal = np.fft.ifft(fourier_coeffs)
time_signal = np.real(time_signal)

time_signal = time_signal / np.max(np.abs(time_signal))

write('output.wav', 48000, time_signal.astype(np.float32))
```

Flag : ZiTF{cdfd75ea44271ef15a3bcb152aa5a412}
