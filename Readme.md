# Sistem Operasi
## Modul 3
## Tugas Sistem Operasi Modul 3

4a. Membuat program untuk melakukan perkalian matriks dengan ukuran matriks pertama 4x2 dan yang kedua 2x5 dan nantinya isi dalam matriks dalam kodingan dan berisi angka 1-20. 

4b. Membuat program yang mengambil hasil dari perkalian 4a kemudian mencari nilai faktorialnya dan tampilkan dalam bentuk array.

4c. Membuat program untuk mengetahui jumlah file dan direktori saat ini dengan emnggunakan IPC pipes.

#4a

Code : 

```#include <stdio.h>
#include <string.h>
#include <pthread.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/ipc.h>
#include <sys/shm.h>
#include <sys/types.h>
#include <sys/wait.h>

#define BELUM_READY -1
#define READY 0
#define SIAP 1
#define MAXIMUM 100

struct shared{
    int status;
    int data[100];
};


int matrixA[4][2];
int matrixb[2][5];
int matrixc[4][5];

int baris = 0;
int kolom = 0;

void* buat_kali(void* arg) {
  if(kolom >= 5){
    kolom = 0;
    baris++;
  }
  for(int i = 0; i < 2;i++){
    matrixc[baris][kolom] += matrixA[baris][i] * matrixb[i][kolom];
  }
  kolom++;
}

int main() {
  srand(time(NULL));
  printf("Matrix A : \n");
  for (int i = 0; i < 4; i++) {
    /* code */
    for (int j = 0; j < 2; j++) {
      matrixA[i][j] = rand()%19+1;
      printf("%2d", matrixA[i][j]);
      /* code */
    }
    printf("\n");
  }

  printf("Matrix B : \n");
  for (int i = 0; i < 2; i++) {
    /* code */
    for (int j = 0; j < 5; j++) {
      /* code */
      matrixb[i][j] = rand()%19+1;
      printf("%2d", matrixb[i][j]);
    }
    printf("\n");
  }

  pthread_t tid[20];

  for (int i = 0; i < 20; i++) {
    /* code */
    pthread_create(&(tid[i]), NULL, &buat_kali, NULL);
  }

  for (int i = 0; i < 20; i++) {
    /* code */
    pthread_join(tid[i], NULL);
  }


  printf("Hasil kali A dan B :\n");
  for (int i = 0; i < 4; i++) {
    /* code */
    for (int j = 0; j < 5; j++) {
      printf("%4d", matrixc[i][j]);
      /* code */
    }
    printf("\n");
  }

   key_t          ShmKEY;
   int            ShmID;
   struct shared  *ShmPTR;

   ShmKEY = ftok("key",100);
   ShmID = shmget(ShmKEY,sizeof(struct shared),IPC_CREAT|0666);
   if(ShmID < 0){
     printf("Shmget error\n");
     exit(1);
   }
   ShmPTR = (struct shared *) shmat(ShmID, NULL, 0);

   ShmPTR->status = BELUM_READY;
   int j = 0;
   int k = 0;

   for(int i = 0; i < 20; i++){
     /* code */
     if(k >= 5){
       /* code */
       j++;
       k = 0;
     }
     ShmPTR->data[i] = matrixc[j][k];
     k++;
   }
    ShmPTR->status = READY;

    printf("Jalanin yang B \n");
    while (ShmPTR->status != SIAP)
        sleep(1);
    printf("B udah jalan\n");
    shmdt((void *) ShmPTR);
    printf("Dah share ke B\n");
    shmctl(ShmID, IPC_RMID, NULL);
    exit(0);
    return 0;
}
```

Penjelasan:

Pada line 1-9 merupakan kode untuk memulai fungsi yang dibutuhkan dalam program ini. Kemudian pada kode
```
#define BELUM_READY -1
#define READY 0
#define SIAP 1
#define MAXIMUM 100
```
berfungsi untuk mengkategorikan elemen angka yang akan dimasukkan sesuai yang tertera yaitu belum_ready jika -1 ready adalah 0 siap adalah 1 dan maximum yaitu 100 . Lalu ada
```
struct shared{
    int status;
    int data[100];
};
``` 
untuk mengkonversi bilangan menjadi bilangan bulat. Kemudian

```
int matrixA[4][2];
int matrixb[2][5];
int matrixc[4][5];

int baris = 0;
int kolom = 0;
```
berfungsi untuk menginputkan matriks yang diminta, jadi kita masukkan dulu matriks a yaitu matriks 4x2 dan matriks b yaitu matriks  2x5 sesuai soal, lalu kita masukkan matriks c yaitu matriks 4x5 sebagai sisa kemungkinan yang ada.
sedangkan kode dibawahnya adalah untuk mendeteksi baris dan kolom. Lalu ada code: 
```
void* buat_kali(void* arg) {
  if(kolom >= 5){
    kolom = 0;
    baris++;
  }
  for(int i = 0; i < 2;i++){
    matrixc[baris][kolom] += matrixA[baris][i] * matrixb[i][kolom];
  }
  kolom++;
}
```
yang pertama untuk perintah void* buat_kali adalah untuk memasukkan fungsi perkalian yang diminta, kemudian kita masukkan perintah if nya dengan menyesuaikan kolom, lalu kita fungsikan for untuk mengalikan matriks a dengan b menggunakan rumus baris dikali kolom. 

```
Kemudian ada kode:
```
int main() {
  srand(time(NULL));
  printf("Matrix A : \n");
  for (int i = 0; i < 4; i++) {
    /* code */
    for (int j = 0; j < 2; j++) {
      matrixA[i][j] = rand()%19+1;
      printf("%2d", matrixA[i][j]);
      /* code */
    }
    printf("\n");
  }
```
kita menggunakan srand() dan rand() untuk mengenerate angka secara acak agar setiap kali dijalankan selalu menghasilkan output yang sama sehingga bisa terwujud dengan apa yang diminta. kemudian setelah itu kita print. Hal itu kita lakukan juga kepada matriks b.
Lalu ada kode
```
  pthread_t tid[20];

  for (int i = 0; i < 20; i++) {
    /* code */
    pthread_create(&(tid[i]), NULL, &buat_kali, NULL);
  }

  for (int i = 0; i < 20; i++) {
    /* code */
    pthread_join(tid[i], NULL);
  }
  ```
  Kita menggunakan pthread_t untuk mendelarasikan variabael, kemudian kita masukkan fungsi for lalu kita buat thread dengan menggunakan funsgi pthread_create jika sudah kita gabungkan semua thread menggunakan pthread_join
  
  lalu ada kode 
  ```
   printf("Hasil kali A dan B :\n");
  for (int i = 0; i < 4; i++) {
    /* code */
    for (int j = 0; j < 5; j++) {
      printf("%4d", matrixc[i][j]);
      /* code */
    }
    printf("\n");
  }
```
Seperti kita semua tahu printf dilakukan untuk mem-print hasil dari kodingan yang sudah kita buat sebelumnya.
Lalu ada 
```
 key_t          ShmKEY;
   int            ShmID;
   struct shared  *ShmPTR;

   ShmKEY = ftok("key",100);
   ShmID = shmget(ShmKEY,sizeof(struct shared),IPC_CREAT|0666);
   if(ShmID < 0){
     printf("Shmget error\n");
     exit(1);
   }
   ```
   digunakan funsgi key antaralain Shmkey, ShmID,ShmPTR untuk menggunakan IPC, jika sudah..
   ```
   ShmPTR = (struct shared *) shmat(ShmID, NULL, 0);

   ShmPTR->status = BELUM_READY;
   int j = 0;
   int k = 0;
   ```
   lalu kita kategorikan dengan define yang telah kita buat sebelumnya. 
   Dan yang terakhir, 
   ```
    for(int i = 0; i < 20; i++){
     /* code */
     if(k >= 5){
       /* code */
       j++;
       k = 0;
     }
     ShmPTR->data[i] = matrixc[j][k];
     k++;
   }
    ShmPTR->status = READY;

    printf("Jalanin yang B \n");
    while (ShmPTR->status != SIAP)
        sleep(1);
    printf("B udah jalan\n");
    shmdt((void *) ShmPTR);
    printf("Dah share ke B\n");
    shmctl(ShmID, IPC_RMID, NULL);
    exit(0);
    return 0;
}
```

kita gunakan fungsi for untuk looping dan gunakan ShmPTR untuk mengeksekusi IPC, kemudian kita print.
   
   
   
   
   
   
   
   
   
   
   
   
















