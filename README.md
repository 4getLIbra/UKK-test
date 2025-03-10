#include <stdio.h>
#include <string.h>
#include <stdlib.h>
#include <time.h>

#define MAX_ITEMS 100

typedef struct {
    char nama[50];
    float harga;
} Barang;

typedef struct {
    char nama[50];
    float harga;
    int jumlah;
    float diskon;
    float subtotal;
} Item;

void bersihkanBuffer() {
    int c;
    while ((c = getchar()) != '\n' && c != EOF); // Membersihkan buffer input
}

void tampilkanBarangTersedia(Barang barangTersedia[], int totalBarang) {
    printf("\n=== DAFTAR BARANG TERSEDIA ===\n");
    for (int i = 0; i < totalBarang; i++) {
        printf("%d. %s - Rp%.2f\n", i + 1, barangTersedia[i].nama, barangTersedia[i].harga);
    }
    printf("\n99. Struk Pembayaran\n");
    printf("55. Reset Pilihan\n");
    printf("00. Keluar\n");
}

void urutkanBarang(Item items[], int totalItems) {
    for (int i = 0; i < totalItems - 1; i++) {
        for (int j = i + 1; j < totalItems; j++) {
            if (items[j].jumlah > items[i].jumlah) { // Urutkan berdasarkan jumlah pembelian terbanyak
                Item temp = items[i];
                items[i] = items[j];
                items[j] = temp;
            }
        }
    }
}

void tampilkanStruk(char kasir[], Item items[], int totalItems, float totalHarga, float totalDiskon) {
    printf("\n===================================================================\n");
    printf("| %-3s | %-6s | %-15s | %-10s | %-10s | %-10s |\n", "No.", "Jumlah", "Nama Barang", "Harga", "Total", "Diskon");
    printf("===================================================================\n");

    for (int i = 0; i < totalItems; i++) {
        printf("| %-3d | %-6d | %-15s | Rp.%-8.2f | Rp.%-8.2f | Rp.%-8.2f |\n",
               i + 1, items[i].jumlah, items[i].nama, items[i].harga,
               items[i].subtotal, items[i].diskon);
    }
    printf("===================================================================\n");
    printf("Total Harga  = Rp. %.2f,-\n", totalHarga);
    printf("Total Diskon = Rp. %.2f,-\n", totalDiskon);
    printf("Total Bayar  = Rp. %.2f,-\n", totalHarga - totalDiskon);
}

void cetakStrukKeFile(char kasir[], Item items[], int totalItems, float totalHarga, float totalDiskon, float uangDibayar, float kembalian) {
    char filename[50];
    time_t now = time(NULL);
    struct tm *t = localtime(&now);
    sprintf(filename, "%04d%02d%02d%02d%02d%02d", t->tm_year + 1900, t->tm_mon + 1, t->tm_mday, t->tm_hour, t->tm_min, t->tm_sec);

    FILE *file = fopen(filename, "w");
    if (file == NULL) {
        printf("Gagal membuka file untuk menulis struk.\n");
        return;
    }

    fprintf(file, "==================================================\n");
    fprintf(file, "|    Toko SKENSA    |\n");
    fprintf(file, "| Jl. MOS Cokroaminoto No. 84, Denpasar |\n");
    fprintf(file, "| Bali | Telp : 0816285791 |\n");
    fprintf(file, "==================================================\n");
    fprintf(file, "ID Struk : %s\n", filename);
    fprintf(file, "==================================================\n");
    fprintf(file, "| Nama Barang        | Harga    | Total    | Diskon    |\n");
    fprintf(file, "==================================================\n");

    for (int i = 0; i < totalItems; i++) {
        char namaBarang[50];
        sprintf(namaBarang, "%dx %s", items[i].jumlah, items[i].nama); // Format: "3x Buku Tulis"
        fprintf(file, "| %-18s | Rp%-6.2f | Rp%-6.2f | Rp%-6.2f |\n",
                namaBarang, items[i].harga, items[i].subtotal, items[i].diskon);
    }

    fprintf(file, "==================================================\n");
    fprintf(file, "Total Harga  = Rp. %.2f,-\n", totalHarga);
    fprintf(file, "Total Diskon = Rp. %.2f,-\n", totalDiskon);
    fprintf(file, "Tagihan      = Rp. %.2f,-\n", totalHarga - totalDiskon);
    fprintf(file, "Pembayaran   = Rp. %.2f,-\n", uangDibayar);
    fprintf(file, "Kembalian    = Rp. %.2f,-\n", kembalian);
    fprintf(file, "==================================================\n");

    // Menampilkan waktu dan tanggal transaksi
    fprintf(file, "Waktu/Hari : %s", asctime(t));
    fprintf(file, "==================================================\n");

    fclose(file);
    printf("Struk telah dicetak ke dalam file %s\n", filename);
}


float hitungDiskon(int jumlah) {
    if (jumlah > 5) {
        return 0.15; // Diskon 15% jika lebih dari 5 item
    } else if (jumlah > 3) {
        return 0.10; // Diskon 10% jika lebih dari 3 item
    }
    return 0; // Tidak ada diskon
}

void resetPilihan(Item items[], int *totalItems, float *subtotal, float *totalDiskon) {
    *totalItems = 0;
    *subtotal = 0;
    *totalDiskon = 0;
    printf("Pilihan telah direset.\n");
}

void prosesPilihanBarang(char input[], Barang barangTersedia[], Item items[], int *totalItems, float *subtotal, float *totalDiskon) {
    char *token = strtok(input, ", ");
    while (token != NULL) {
        int pilihanBarang = atoi(token);

        if (pilihanBarang < 1 || pilihanBarang > 5) {
            printf("Pilihan %d tidak valid!\n", pilihanBarang);
        } else {
            int indexBarang = pilihanBarang - 1;
            items[*totalItems].harga = barangTersedia[indexBarang].harga;
            strcpy(items[*totalItems].nama, barangTersedia[indexBarang].nama);

            printf("Masukkan jumlah barang %s: ", barangTersedia[indexBarang].nama);
            scanf("%d", &items[*totalItems].jumlah);
            bersihkanBuffer(); // Membersihkan buffer setelah scanf

            items[*totalItems].diskon = hitungDiskon(items[*totalItems].jumlah) * items[*totalItems].harga * items[*totalItems].jumlah;
            items[*totalItems].subtotal = items[*totalItems].harga * items[*totalItems].jumlah;

            *subtotal += items[*totalItems].subtotal;
            *totalDiskon += items[*totalItems].diskon;
            (*totalItems)++;
        }

        token = strtok(NULL, ", ");
    }
}

int main() {
    Barang barangTersedia[] = {
        {"Buku Tulis", 5000},
        {"Pensil", 2000},
        {"Penghapus", 1000},
        {"Penggaris", 1000},
        {"Bujur Sangkar", 500}
    };
    int totalBarangTersedia = sizeof(barangTersedia) / sizeof(barangTersedia[0]);

    Item items[MAX_ITEMS];
    int totalItems = 0;
    char inputBarang[100], tambahLagi;
    float subtotal = 0, totalDiskon = 0;
    float uangDibayar = 0, kembalian = 0;
    char kasir[50];

    printf("=== SISTEM KASIR ===\n");
    printf("Masukkan nama kasir: ");
    fgets(kasir, sizeof(kasir), stdin);
    kasir[strcspn(kasir, "\n")] = 0;

    while (1) { // Perulangan utama
        tampilkanBarangTersedia(barangTersedia, totalBarangTersedia);
        printf("Pilih barang (misalnya: 1, 2, 3): ");
        fgets(inputBarang, sizeof(inputBarang), stdin);
        inputBarang[strcspn(inputBarang, "\n")] = 0;

        if (strcmp(inputBarang, "99") == 0) {
            if (totalItems == 0) {
                printf("Belum ada barang yang dipilih.\n");
                continue;
            }
            urutkanBarang(items, totalItems);
            tampilkanStruk(kasir, items, totalItems, subtotal, totalDiskon);

            do {
                printf("Masukkan uang bayar: ");
                scanf("%f", &uangDibayar);
                bersihkanBuffer(); // Membersihkan buffer setelah scanf
                if (uangDibayar < (subtotal - totalDiskon)) {
                    printf("Uang yang dibayarkan kurang. Silakan masukkan jumlah yang cukup.\n");
                }
            } while (uangDibayar < (subtotal - totalDiskon));

            kembalian = uangDibayar - (subtotal - totalDiskon);
            printf("Kembalian: Rp. %.2f,-\n", kembalian);

            cetakStrukKeFile(kasir, items, totalItems, subtotal, totalDiskon, uangDibayar, kembalian);
            resetPilihan(items, &totalItems, &subtotal, &totalDiskon);
        } else if (strcmp(inputBarang, "55") == 0) {
            resetPilihan(items, &totalItems, &subtotal, &totalDiskon);
        } else if (strcmp(inputBarang, "00") == 0) {
            printf("Terima kasih telah menggunakan sistem kasir.\n");
            break;
        } else {
            prosesPilihanBarang(inputBarang, barangTersedia, items, &totalItems, &subtotal, &totalDiskon);
        }

        printf("Tambah barang lagi? (y/n): ");
        scanf(" %c", &tambahLagi);
        bersihkanBuffer(); // Membersihkan buffer setelah scanf

        if (tambahLagi == 'n' || tambahLagi == 'N') {
            // Jika pengguna memilih "n", tampilkan opsi struk, reset, atau keluar
            printf("\nPilih opsi:\n");
            printf("99. Struk Pembayaran\n");
            printf("55. Reset Pilihan\n");
            printf("00. Keluar\n");
            printf("Pilihan Anda: ");
            fgets(inputBarang, sizeof(inputBarang), stdin);
            inputBarang[strcspn(inputBarang, "\n")] = 0;

            if (strcmp(inputBarang, "99") == 0) {
                if (totalItems == 0) {
                    printf("Belum ada barang yang dipilih.\n");
                    continue;
                }
                urutkanBarang(items, totalItems);
                tampilkanStruk(kasir, items, totalItems, subtotal, totalDiskon);

                do {
                    printf("Masukkan uang bayar: ");
                    scanf("%f", &uangDibayar);
                    bersihkanBuffer(); // Membersihkan buffer setelah scanf
                    if (uangDibayar < (subtotal - totalDiskon)) {
                        printf("Uang yang dibayarkan kurang. Silakan masukkan jumlah yang cukup.\n");
                    }
                } while (uangDibayar < (subtotal - totalDiskon));

                kembalian = uangDibayar - (subtotal - totalDiskon);
                printf("Kembalian: Rp. %.2f,-\n", kembalian);

                cetakStrukKeFile(kasir, items, totalItems, subtotal, totalDiskon, uangDibayar, kembalian);
                resetPilihan(items, &totalItems, &subtotal, &totalDiskon);
            } else if (strcmp(inputBarang, "55") == 0) {
                resetPilihan(items, &totalItems, &subtotal, &totalDiskon);
            } else if (strcmp(inputBarang, "00") == 0) {
                printf("Terima kasih telah menggunakan sistem kasir.\n");
                break;
            }
        }
    }

    return 0;
}
