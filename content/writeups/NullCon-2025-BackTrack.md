---
title: "NullCon 2025 : BackTrack | Revese Engineering"
date: 2025-03-25
draft: false
categories: ["Reverse"]
ctf: "NullCon 2025"
challenge: "backtrack"
author: "Reyko"
---


# Write‑up du Challenge "BackTrack"

## Contexte et Énoncé : 
**While Working I found an interesting Sample in the wild. The Sample seems to load other executables but idk how? can you find that out?
WARNING: THIS IS AN ACTUAL MALWARE, DO NOT EXECUTE IT (the challenge is meant to be solved statically)
ALSO I OVERWROTE A PART OF THE ENTRY TO MAKE THE SAMPLE NOT EXECUTABLE (just to be extra safe ;) ) Files can be found in the zip:

b8ad5cbf8c8a3129582161226e79b6c4b67c8b868592d1618252451c8c2146c8 is the sample
input_file is the data they used to get the keylogger
keylogger.bin is the result after the input_file is given to the sample
data.bin is the challenge file
To solve this task reverse what's in data.bin and look at the output (example: input_file -> keylogger, data.bin -> ???)

Le défi est donc de comprendre comment ce malware (que l’on appellera BackTrack) charge d’autres exécutables – et de découvrir le contenu caché dans data.bin. Spoiler : il s’agit d’un flux compressé qui, une fois décompressé, révèle une image PE (probablement une DLL keylogger).

### 1. Schéma Global du Malware
Le malware se la pète en plusieurs étapes :

Initialisation et Protection

sub_401500 (Instance/Initialization)
C’est le videur de boîte qui s’assure qu’il n’y a qu’un seul exemplaire qui traîne sur le système. Il charge aussi quelques API en mode furtif, notamment pour récupérer SHGetValueA (un petit coup de pouce pour lire des données plus tard).
sub_401800 (Anti-Sandbox Delay)
Un délai bien calculé pour embrouiller les sandboxes et débogueurs – parce qu’on ne veut pas que nos amis analytiques aient trop le temps de respirer !
Extraction et Décompression du Payload

sub_401330 (Data Extraction)
Le malware fouille dans le registre (ou autre) pour récupérer un flux compressé, en utilisant la fameuse clé "Software\Realtek Inc." (oui, c’est là que ça se passe).
sub_401690 (Decompression Engine)
Ici, c’est l’usine à décompression qui transforme ce flux compressé en une image claire. En mode LZSS, chaque bit de contrôle détermine si on copie un octet directement ou si on va fouiller dans le buffer déjà décompressé pour recopier une séquence. Si le premier octet vaut 1, c’est le mode "copie simple" qui s’active.
Chargement du Module PE

sub_4025d0 (Wrapper) et sub_402600 (PE Loader)
Une fois décompressé, le buffer est envoyé à l’atelier de montage qui vérifie que le fichier est bien un PE (MZ, PE, etc.), alloue la mémoire nécessaire, et copie les sections. Bref, c’est le montage final du keylogger !
Exécution du Payload

sub_4023b0 (Entry Point Resolver)
Le malware reconstruit la fameuse chaîne "DllMain" (en collant "Dll", "Ma", "in") pour trouver le point d’entrée de la DLL et lancer le payload dans un nouveau thread. Le show commence !
Nettoyage

sub_4022b0 (Cleanup)
Et si quelque chose cloche, ou simplement pour faire bien pro, le malware nettoie tout le bazar en libérant les ressources allouées.

### 2. Détail des Fonctions
#### 2.1. main (Entry Point)
Code représentatif :

    {
        if (!sub_401500())
        {
            sub_401800();
            int32_t var_8_1 = 0;
            sub_401180();
        }
        
        return 0;
    }

Ce qu’elle fait :

main (Entry Point) lance l’aventure.
Il appelle sub_401500 (Instance/Initialization) pour s’assurer qu’on ne va pas avoir deux versions qui se marchent sur les pieds.
Ensuite, sub_401800 (Anti-Sandbox Delay) met en place un petit délai pour dérouter les outils d’analyse automatisés.
Enfin, sub_401180 (Payload Launcher) se charge de démarrer la chaîne de récupération et d’exécution du payload.

#### 2.2. sub_401500 (Instance/Initialization)
Code représentatif :

int32_t sub_401500() {
    HANDLE hObject = CreateMutexA(nullptr, 0, "SoftUpdateApp");
    int32_t result;
    
    if (GetLastError() != ERROR_ALREADY_EXISTS) {
        // Charge dynamiquement "kernel32.dll" et "SHLWAPI.dll"
        // Récupère l'adresse de SHGetValueA
        result = 0;
    } else {
        CloseHandle(hObject);
        result = 0xffffffff;
    }
    
    return result;
}

Ce qu’elle fait :

Crée un mutex nommé "SoftUpdateApp" pour vérifier qu’une seule instance du malware tourne.
Si une autre instance est détectée, il se ferme gracieusement ; sinon, il charge en douceur les bibliothèques nécessaires pour la suite.

#### 2.3. sub_401800 (Anti-Sandbox Delay)
Code représentatif :

uint32_t sub_401800() {
    GetLocalTime(&lpSystemTime_1);
    int16_t var_c;  // On récupère un bout d'heure (ex. les secondes)
    uint32_t ecx = (uint32_t)var_c;
    int32_t var_30_1;
    
    if (ecx > 0x2d)
        var_30_1 = ecx - 0x2d;
    else if (ecx != 0x2d)
        var_30_1 = ecx + 0xe;
    else
        var_30_1 = ecx - 0x1e;
    
    uint32_t var_34 = GetTickCount();
    int32_t var_3c = 0xafc8;
    int32_t var_40 = 0x57e40;
    uint32_t i;
    
    do {
        GetLocalTime(&lpSystemTime);
        i = GetTickCount() - var_34;
        int16_t var_1c;
        if (i > 0xafc8 && (uint32_t)var_1c == var_30_1)
            break;
    } while (i <= 0x57e40);
    
    return i;
}
Ce qu’elle fait :

Calcule un délai à partir de l’heure locale et du tick système.
L’idée ? Retarder l’exécution pour embrouiller les environnements de sandbox ou les débogueurs qui scrutent chaque microseconde.

#### 2.4. sub_401180 (Payload Launcher)
Code représentatif :

int32_t sub_401180() {
    char* eax_3 = sub_401330(&var_40);
    int32_t* eax_4 = sub_4025d0(eax_3, var_40);
    
    int32_t var_34;
    memset(&var_34, 0, 0x1c);
    strcat(&var_34, "Dll");
    strcat(&var_34, "Ma");
    strcat(&var_34, "in");
    
    int32_t var_44 = sub_4023b0(eax_4, &var_34);
    
    HANDLE hThread = CreateThread(nullptr, 0, sub_4023b0(eax_4, &var_34), nullptr, THREAD_CREATE_RUN_IMMEDIATELY, nullptr);
    WaitForSingleObject(hThread, INFINITE);
    
    Sleep(0xbb8);
    sub_4022b0(eax_4);
    if (eax_3)
        free(eax_3);
    
    return 0;
}
Ce qu’elle fait :

Lance le processus de récupération du payload.
Appelle sub_401330 (Data Extraction) pour extraire le flux compressé.
Le passe ensuite à sub_4025d0 (Wrapper), qui envoie le tout à sub_402600 (PE Loader) pour monter le module en mémoire.
Reconstruit la chaîne "DllMain" pour trouver le point d’entrée et lance le payload dans un nouveau thread.
Nettoie les ressources à la fin.

#### 2.5. sub_401330 (Data Extraction)
Code représentatif :

char* sub_401330(uint32_t* arg1) {
    char var_30[0x16];
    strncpy(var_30, "Software\\Realtek Inc.", 0x16);
    
    uint32_t _Size_1 = 0, _Size_2 = 4;
    int32_t var_48 = 3;
    
    if (!data_407058(0x80000001, &var_30, &data_405140, &var_48, &_Size_1, &_Size_2)) {
        char* buffer = malloc(_Size_1);
        _Size_2 = _Size_1;
        if (!data_407058(0x80000001, &var_30, &data_405144, &var_48, buffer, &_Size_2)) {
            uint32_t _Size = _Size_1 << 1;
            char* resultBuffer = malloc(_Size);
            if (resultBuffer) {
                sub_401690(buffer, _Size_1, resultBuffer, arg1);
                free(buffer);
                return resultBuffer;
            }
        } else {
            return NULL;
        }
    } else {
        return NULL;
    }
    return NULL;
}
Ce qu’elle fait :

Fouille dans le registre (ou un mécanisme équivalent) pour récupérer un flux de données compressé à partir de la clé "Software\Realtek Inc.".
Effectue un appel pour obtenir la taille, puis un autre pour récupérer le contenu dans un buffer.
Envoie ensuite ce buffer à sub_401690 (Decompression Engine) pour le décompresser.

#### 2.6. sub_401690 (Decompression Engine)
Code représentatif (implémentation en C) :

int32_t* sub_401690(char* arg1, int32_t arg2, char* arg3, int32_t* arg4) {
    if ((uint8_t)arg1[0] != 1) {
        char *in_ptr = arg1 + 4;  // On saute l'en-tête de 4 octets
        char *in_end = arg1 + arg2;
        char *out_ptr = arg3;
        
        uint16_t controlBits = 0;
        int bitCount = 0;
        
        while (in_ptr < in_end) {
            if (bitCount == 0) {
                // Lecture d'un bloc de 16 bits de contrôle (little-endian)
                controlBits = ((uint8_t)in_ptr[1] << 8) | (uint8_t)in_ptr[0];
                in_ptr += 2;
                bitCount = 16;
            }
            
            if ((controlBits & 1) == 0) {
                *out_ptr++ = *in_ptr++;
            } else {
                int length = ((uint8_t)in_ptr[0] & 0x0F) + 1;
                int offset = (uint8_t)in_ptr[1] + (((uint8_t)in_ptr[0] & 0xF0) << 4);
                in_ptr += 2;
                char *src = out_ptr - offset;
                for (int j = 0; j < length; j++) {
                    *out_ptr++ = src[j];
                }
            }
            controlBits >>= 1;
            bitCount--;
        }
        *arg4 = (int32_t)(out_ptr - arg3);
    } else {
        memcpy(arg3, arg1 + 4, arg2 - 4);
        *arg4 = arg2 - 4;
    }
    return arg4;
}
Ce qu’elle fait :

Si le premier octet du flux compressé n’est pas égal à 1, elle utilise un algorithme de décompression de type LZSS :
Ignore les 4 octets d’en-tête.
Lit un bloc de 16 bits de contrôle pour déterminer, bit par bit, si l’octet suivant est copié directement ou si une séquence doit être copiée depuis une partie antérieure du buffer décompressé.
Pour un bit à 1, deux octets donnent la longueur (les 4 bits faibles + 1) et l’offset (les 4 bits forts combinés avec le second octet).
Si le premier octet vaut 1, il effectue simplement une copie du reste du buffer.
Le résultat est le buffer décompressé et sa taille, renvoyés via le paramètre arg4.

#### 2.7. sub_4025d0 (Wrapper)
Code représentatif :

int32_t* sub_4025d0(int16_t* arg1, int32_t arg2) {
    return sub_402600(arg1, arg2, ___crtGetLocaleInfoEx, ___vcrt_LoadLibraryExW, sub_402280, __beep, __seterrormode, 0);
}
Ce qu’elle fait :

Sert de simple intermédiaire pour transmettre le buffer décompressé à sub_402600 (PE Loader), en passant également quelques fonctions de la CRT et API Windows nécessaires pour l’allocation mémoire et d’autres opérations.

#### 2.8. sub_402600 (PE Loader)
Code représentatif :

int32_t* sub_402600(int16_t* arg1, int32_t arg2, int32_t arg3, int32_t arg4, int32_t arg5, int32_t arg6, int32_t arg7, int32_t arg8) {
    int32_t var_24 = 0;
    
    if (!sub_401b70(arg2, 0x40))
        return NULL;
    
    if ((uint16_t)*arg1 != 0x5a4d) {
        SetLastError(ERROR_BAD_EXE_FORMAT);
        return NULL;
    }
    
    if (!sub_401b70(arg2, *(uint32_t*)((char*)arg1 + 0x3c) + 0xf8))
        return NULL;
    
    int32_t* edx_5 = (char*)arg1 + *(uint32_t*)((char*)arg1 + 0x3c);
    
    if (*(uint32_t*)edx_5 != 0x4550) {
        SetLastError(ERROR_BAD_EXE_FORMAT);
        return NULL;
    }
    
    if ((uint32_t)edx_5[1] != 0x14c) {
        SetLastError(ERROR_BAD_EXE_FORMAT);
        return NULL;
    }
    
    if (edx_5[0xe] & 1) {
        SetLastError(ERROR_BAD_EXE_FORMAT);
        return NULL;
    }
    
    void* var_14_1 = (char*)edx_5 + (uint32_t)edx_5[5] + 0x18;
    for (int i = 0; i < *(uint16_t*)((char*)edx_5 + 6); i++) {
        int32_t var_20_1;
        if (*(uint32_t*)((char*)var_14_1 + 0x10))
            var_20_1 = *(uint32_t*)((char*)var_14_1 + 0xc) + *(uint32_t*)((char*)var_14_1 + 0x10);
        else
            var_20_1 = *(uint32_t*)((char*)var_14_1 + 0xc) + edx_5[0xe];
        if (var_20_1 > var_24)
            var_24 = var_20_1;
        var_14_1 = (char*)var_14_1 + 0x28;
    }
    
    int32_t var_60;
    GetNativeSystemInfo(&var_60);
    int32_t eax_16 = sub_401920(edx_5[0x14], var_60);
    if (eax_16 != sub_401920(var_24, var_60)) {
        SetLastError(ERROR_BAD_EXE_FORMAT);
        return NULL;
    }
    
    int32_t var_10_1 = arg3(edx_5[0xd], eax_16, 0x3000, 4, arg8);
    if (!var_10_1) {
        var_10_1 = arg3(0, eax_16, 0x3000, 4, arg8);
        if (!var_10_1) {
            SetLastError(ERROR_OUTOFMEMORY);
            return NULL;
        }
    }
    
    int32_t* result = HeapAlloc(GetProcessHeap(), HEAP_ZERO_MEMORY, 0x40);
    if (!result) {
        arg4(var_10_1, 0, 0x8000, arg8);
        SetLastError(ERROR_OUTOFMEMORY);
        return NULL;
    }
    
    result[1] = var_10_1;
    int32_t var_2c_1 = ((uint16_t)*((char*)edx_5 + 0x16) & 0x2000) ? 1 : 0;
    result[5] = var_2c_1;
    result[7] = arg3;
    result[8] = arg4;
    result[9] = arg5;
    result[0xa] = arg6;
    result[0xb] = arg7;
    result[0xd] = arg8;
    result[0xf] = var_60;
    
    if (sub_401b70(arg2, edx_5[0x15])) {
        int32_t eax_35 = arg3(var_10_1, edx_5[0x15], 0x1000, 4, arg8);
        memcpy(eax_35, arg1, edx_5[0x15]);
        *(uint32_t*)result = eax_35 + *(uint32_t*)((char*)arg1 + 0x3c);
        *(uint32_t*)(*(uint32_t*)result + 0x34) = var_10_1;
        if (sub_401b90(arg1, arg2, edx_5, result)) {
            int32_t eax_43 = *(uint32_t*)(*(uint32_t*)result + 0x34);
            if (eax_43 == edx_5[0xd])
                result[6] = 1;
            else
                result[6] = sub_4020d0(result, eax_43 - edx_5[0xd]);
            if (sub_401940(result) && sub_401ee0(result)) {
                sub_401d00(result);
                if (!*(uint32_t*)(*(uint32_t*)result + 0x28)) {
                    result[0xe] = 0;
                    return result;
                }
                if (!result[5]) {
                    result[0xe] = var_10_1 + *(uint32_t*)(*(uint32_t*)result + 0x28);
                    return result;
                }
                if ((var_10_1 + *(uint32_t*)(*(uint32_t*)result + 0x28))(var_10_1, 1, 0)) {
                    result[4] = 1;
                    return result;
                }
                SetLastError(ERROR_DLL_INIT_FAILED);
            }
        }
    }
    
    sub_4022b0(result);
    return NULL;
}
Ce qu’elle fait :

Vérifie minutieusement que le fichier décompressé est un vrai fichier PE (en contrôlant les signatures "MZ" et "PE", le type de machine, etc.).
Parcourt la table des sections pour déterminer la taille maximale requise et s’assure de la compatibilité avec l’architecture système.
Alloue la mémoire nécessaire pour charger le module, copie l’en‑tête et les sections, et résout les relocations/imports via d’autres fonctions spécialisées.
Crée une structure de contrôle pour gérer le module.
En cas de pépin, appelle sub_4022b0 (Cleanup) pour libérer les ressources.

#### 2.9. sub_4012f0 (Simple Copier)
Code représentatif :

int32_t sub_4012f0(char* arg1, char* arg2, int32_t arg3) {
    while (arg3--) {
        *arg2++ = *arg1++;
    }
    return 0;
}
Ce qu’elle fait :

Copie simplement arg3 octets de arg1 vers arg2.
Utilisée en mode copie simple dans sub_401690 (Decompression Engine) lorsque le flux n’est pas compressé.

### 3. Récupération de l’Image Contenue dans data.bin
Les indices sont clairs :

data.bin contient un flux compressé.
sub_401690 (Decompression Engine) décompresse ce flux pour produire une image PE, qui est ensuite montée en mémoire par le malware.
Pour résoudre le challenge, nous avons implémenté en C la fonction sub_401690. Le fichier source (nommé ici solve.c) lit data.bin, décompresse son contenu et sauvegarde le résultat dans recovered_image.bin.

#### Implémentation en C pour Récupérer l’Image
```
#include <stdint.h>
#include <stdlib.h>
#include <string.h>
#include <stdio.h>

// Implémentation de sub_401690 (Decompression Engine)
int32_t* sub_401690(char* arg1, int32_t arg2, char* arg3, int32_t* arg4) {
    if ((uint8_t)arg1[0] != 1) {
        char *in_ptr = arg1 + 4;  // Saut de l'en-tête de 4 octets
        char *in_end = arg1 + arg2;
        char *out_ptr = arg3;
        
        uint16_t controlBits = 0;
        int bitCount = 0;
        
        while (in_ptr < in_end) {
            if (bitCount == 0) {
                controlBits = ((uint8_t)in_ptr[1] << 8) | (uint8_t)in_ptr[0];
                in_ptr += 2;
                bitCount = 16;
            }
            
            if ((controlBits & 1) == 0) {
                *out_ptr++ = *in_ptr++;
            } else {
                int length = ((uint8_t)in_ptr[0] & 0x0F) + 1;
                int offset = (uint8_t)in_ptr[1] + (((uint8_t)in_ptr[0] & 0xF0) << 4);
                in_ptr += 2;
                char *src = out_ptr - offset;
                for (int j = 0; j < length; j++) {
                    *out_ptr++ = src[j];
                }
            }
            controlBits >>= 1;
            bitCount--;
        }
        *arg4 = (int32_t)(out_ptr - arg3);
    } else {
        memcpy(arg3, arg1 + 4, arg2 - 4);
        *arg4 = arg2 - 4;
    }
    return arg4;
}

#ifdef TEST_SUB_401690
int main(void) {
    // Ouvre data.bin en mode binaire
    FILE *f = fopen("data.bin", "rb");
    if (!f) {
        fprintf(stderr, "Erreur d'ouverture de data.bin\n");
        return 1;
    }
    
    // Détermine la taille du fichier
    fseek(f, 0, SEEK_END);
    long file_size = ftell(f);
    fseek(f, 0, SEEK_SET);
    
    if (file_size <= 0) {
        fprintf(stderr, "Fichier vide ou erreur de taille\n");
        fclose(f);
        return 1;
    }
    
    // Alloue un buffer pour le fichier compressé
    char *compressed_data = malloc(file_size);
    if (!compressed_data) {
        fprintf(stderr, "Erreur d'allocation pour compressed_data\n");
        fclose(f);
        return 1;
    }
    
    // Lit le fichier dans le buffer
    if (fread(compressed_data, 1, file_size, f) != (size_t)file_size) {
        fprintf(stderr, "Erreur de lecture du fichier\n");
        free(compressed_data);
        fclose(f);
        return 1;
    }
    fclose(f);
    
    // Alloue un buffer de sortie (taille estimée : 2x la taille compressée)
    char *decompressed_buffer = malloc(file_size * 2);
    if (!decompressed_buffer) {
        fprintf(stderr, "Erreur d'allocation pour decompressed_buffer\n");
        free(compressed_data);
        return 1;
    }
    
    int32_t decompressed_size = 0;
    sub_401690(compressed_data, file_size, decompressed_buffer, &decompressed_size);
    
    printf("Taille décompressée : %d\n", decompressed_size);
    
    // Sauvegarde le résultat dans recovered_image.bin
    FILE *out = fopen("recovered_image.bin", "wb");
    if (!out) {
        fprintf(stderr, "Erreur d'ouverture du fichier de sortie\n");
        free(compressed_data);
        free(decompressed_buffer);
        return 1;
    }
    fwrite(decompressed_buffer, 1, decompressed_size, out);
    fclose(out);
    
    free(compressed_data);
    free(decompressed_buffer);
    
    return 0;
}
#endif
```

Commande de Compilation
Si le fichier est nommé solve.c, compilez-le avec :

gcc -DTEST_SUB_401690 -o solve solve.c
Puis, lancez l’exécutable :

./solve
Cela générera recovered_image.bin, qui contient l’image décompressée (votre futur objet d’analyse avec vos outils favoris comme IDA Pro ou Ghidra).

### 4. Conclusion
En résumé, le challenge BackTrack nous a montré comment ce malware super malin :

Instance/Initialization (sub_401500) – S’assure qu’il est le seul à faire la bringue sur le système et charge ses outils secrets.
Anti-Sandbox Delay (sub_401800) – Introduit un délai pour embrouiller ceux qui voudraient l’analyser.
Data Extraction (sub_401330) – Récupère un flux compressé caché dans le registre (ou ailleurs).
Decompression Engine (sub_401690) – Décompresse ce flux à la façon LZSS pour en révéler une image PE.
Wrapper (sub_4025d0) & PE Loader (sub_402600) – Montent le module en mémoire, en vérifiant que tout est en ordre.
Entry Point Resolver (sub_4023b0) – Trouve le fameux "DllMain" et lance le payload dans un thread.
Cleanup (sub_4022b0) – Nettoie le bazar après le show.


WU made with <3 with the help of my bestbro openai
