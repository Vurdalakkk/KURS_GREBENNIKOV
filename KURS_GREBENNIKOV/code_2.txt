#define _CRT_SECURE_NO_WARNINGS
#include <stdio.h>
#include <locale.h>
#include <stdlib.h>
#include <string.h>
#include <ctype.h>

// КОНСТАНТЫ И СТРУКТУРЫ

#define MAX_NAME_LEN 25
#define MAX_OS_LEN 20
#define MAX_FAMILY_LEN 20
#define MAX_GPU_MODEL_LEN 30
#define FILENAME_LEN 50
#define MAX_RECORDS 100

// Структура для хранения данных о компьютере
typedef struct {
    char processor[MAX_NAME_LEN];          // Процессор
    int ram_size;                          // Объем оперативной памяти (ГБ)
    char manufacturer[MAX_NAME_LEN];       // Производитель компьютера
    int is_gaming;                         // Игровой компьютер (1-да, 0-нет)
    char cpu_family[MAX_FAMILY_LEN];       // Семейство процессоров
    char gpu_model[MAX_GPU_MODEL_LEN];     // Модель дискретной видеокарты
    char os[MAX_OS_LEN];                   // Операционная система
    int ssd_size;                          // Объем SSD (ГБ)
    int vram_size;                         // Объем видеопамяти (ГБ)
    float cpu_clock_speed;                 // Тактовая частота процессора (ГГц)
} Computer;

// ПРОТОТИПЫ ФУНКЦИЙ

// Основные функции программы
void print_welcome_message(void);
void fill_computer_data(Computer* pc);
void print_all_computers(Computer* computers, int count);
void search_in_file(void);
void save_to_file(const char* filename, Computer* computers, int count);
int load_from_file(const char* filename, Computer* computers, int max_count);
void sort_and_print_from_file(void);
void modify_record_in_file(void);
void add_records_to_file(void);
void view_file_contents(void);

// Вспомогательные функции
void clear_input_buffer(void);
int get_valid_int(const char* prompt, int min_val);
float get_valid_float(const char* prompt, float min_val);
int get_yes_no(const char* prompt);
void print_computer_table(Computer* computers, int count);
void trim_string(char* str);
void print_search_results(Computer* computers, int count, const char* message);  // ДОБАВЛЕНО

// Функции сравнения для сортировки 
int compare_processor(const void* a, const void* b);
int compare_gaming(const void* a, const void* b);

// MAIN 

int main() {
    setlocale(LC_ALL, "RUS");

    Computer computers[MAX_RECORDS];
    int count = 0;
    int choice;

    print_welcome_message();

    do {
        printf("\n=== ГЛАВНОЕ МЕНЮ ===\n");
        printf("1. Создание и сохранение новой записи\n");
        printf("2. Поиск записи по заданным полям\n");
        printf("3. Работа с файлами (сохранение/загрузка/просмотр)\n");
        printf("4. Печать всех записей с сортировкой\n");
        printf("5. Изменение записи в файле\n");
        printf("6. Добавление новых записей в файл\n");
        printf("0. Выход\n");
        printf("Выберите действие (0-6): ");

        scanf("%d", &choice);
        clear_input_buffer();

        switch (choice) {
        case 1: {
            int num_records;
            printf("Сколько записей создать? (макс. %d): ", MAX_RECORDS);
            scanf("%d", &num_records);
            clear_input_buffer();

            if (num_records <= 0 || num_records > MAX_RECORDS) {
                printf("Некорректное количество записей!\n");
                break;
            }

            for (int i = 0; i < num_records; i++) {
                printf("\n=== Запись %d из %d ===\n", i + 1, num_records);
                fill_computer_data(&computers[i]);
            }
            count = num_records;

            printf("\n=== Созданные записи ===\n");
            print_all_computers(computers, count);

            // Сразу предлагаем сохранить
            printf("\nСохранить созданные записи в файл? (y/n): ");
            char save_choice;
            scanf(" %c", &save_choice);
            clear_input_buffer();

            if (save_choice == 'y' || save_choice == 'Y') {
                char filename[FILENAME_LEN];
                printf("Введите имя файла для сохранения: ");
                fgets(filename, FILENAME_LEN, stdin);
                trim_string(filename);
                save_to_file(filename, computers, count);
                printf("Записи сохранены в файл '%s'\n", filename);
            }
            break;
        }

        case 2: {
            // Поиск работает непосредственно в файле
            search_in_file();
            break;
        }

        case 3: {
            int file_choice;
            printf("\n=== РАБОТА С ФАЙЛАМИ ===\n");
            printf("1. Сохранить текущие данные из памяти в файл\n");
            printf("2. Загрузить данные из файла в память\n");
            printf("3. Просмотреть содержимое файла (без загрузки в память)\n");
            printf("Выберите действие (1-3): ");
            scanf("%d", &file_choice);
            clear_input_buffer();

            if (file_choice == 1) {
                if (count == 0) {
                    printf("Нет данных в памяти для сохранения!\n");
                    break;
                }

                char filename[FILENAME_LEN];
                printf("Введите имя файла для сохранения: ");
                fgets(filename, FILENAME_LEN, stdin);
                trim_string(filename);
                save_to_file(filename, computers, count);
                printf("Данные сохранены в файл '%s'\n", filename);
            }
            else if (file_choice == 2) {
                char filename[FILENAME_LEN];
                printf("Введите имя файла для загрузки: ");
                fgets(filename, FILENAME_LEN, stdin);
                trim_string(filename);
                count = load_from_file(filename, computers, MAX_RECORDS);
                if (count > 0) {
                    printf("Загружено %d записей в память\n", count);
                    print_all_computers(computers, count);
                }
            }
            else if (file_choice == 3) {
                view_file_contents();
            }
            else {
                printf("Неверный выбор!\n");
            }
            break;
        }

        case 4: {
            // Сортировка работает непосредственно с файлом
            sort_and_print_from_file();
            break;
        }

        case 5: {
            modify_record_in_file();
            break;
        }

        case 6: {
            add_records_to_file();
            break;
        }

        case 0:
            printf("Выход из программы...\n");
            break;

        default:
            printf("Неверный выбор! Введите число от 0 до 6.\n");
        }
    } while (choice != 0);

    return 0;
}

// ОПРЕДЕЛЕНИЯ ФУНКЦИЙ 

void print_welcome_message(void) {
    printf("=============================================\n");
    printf("  ПРОГРАММА БАЗЫ ДАННЫХ ПЕРСОНАЛЬНЫХ КОМПЬЮТЕРОВ\n");
    printf("=============================================\n");
    printf("Программа предназначена для работы с записями\n");
    printf("данных о персональных компьютерах:\n");
    printf("- Создание и редактирование записей\n");
    printf("- Поиск по процессору и объему ОЗУ\n");
    printf("- Сортировка по процессору и игровому статусу\n");
    printf("- Сохранение и загрузка из файлов\n");
    printf("=============================================\n\n");
}

void clear_input_buffer(void) {
    int c;
    while ((c = getchar()) != '\n' && c != EOF);
}

int get_valid_int(const char* prompt, int min_val) {
    int value;
    while (1) {
        printf("%s", prompt);
        if (scanf("%d", &value) == 1 && value >= min_val) {
            clear_input_buffer();
            return value;
        }
        printf("Ошибка! Введите целое число >= %d\n", min_val);
        clear_input_buffer();
    }
}

float get_valid_float(const char* prompt, float min_val) {
    float value;
    while (1) {
        printf("%s", prompt);
        if (scanf("%f", &value) == 1 && value >= min_val) {
            clear_input_buffer();
            return value;
        }
        printf("Ошибка! Введите число >= %.1f\n", min_val);
        clear_input_buffer();
    }
}

int get_yes_no(const char* prompt) {
    char input;
    while (1) {
        printf("%s (y/n): ", prompt);
        scanf(" %c", &input);
        clear_input_buffer();

        if (input == 'y' || input == 'Y')
            return 1;
        else if (input == 'n' || input == 'N')
            return 0;
        else
            printf("Ошибка! Введите 'y' или 'n'\n");
    }
}

void trim_string(char* str) {
    int i = strlen(str) - 1;
    while (i >= 0 && isspace((unsigned char)str[i])) {
        str[i] = '\0';
        i--;
    }
}

void print_search_results(Computer* computers, int count, const char* message) {
    if (count == 0) {
        printf("\n+---------------------------------------------------------------------------------------------------------------------------+\n");
        printf("|                                              ЗАПИСИ НЕ НАЙДЕНЫ                                                              |\n");
        printf("+---------------------------------------------------------------------------------------------------------------------------+\n");
        return;
    }

    printf("\n%s\n", message);
    print_computer_table(computers, count);
    printf("Найдено записей: %d\n", count);
}

// Функция для просмотра содержимого файла
void view_file_contents(void) {
    char filename[FILENAME_LEN];
    printf("Введите имя файла для просмотра: ");
    fgets(filename, FILENAME_LEN, stdin);
    trim_string(filename);

    Computer temp_computers[MAX_RECORDS];
    int temp_count = load_from_file(filename, temp_computers, MAX_RECORDS);

    if (temp_count == 0) {
        printf("Файл пуст, не существует или содержит ошибки!\n");
        return;
    }

    printf("\n=== СОДЕРЖИМОЕ ФАЙЛА '%s' ===\n", filename);
    print_all_computers(temp_computers, temp_count);
    printf("Всего записей в файле: %d\n", temp_count);
}

void fill_computer_data(Computer* pc) {
    printf("Введите процессор (например, Intel Core i5): ");
    fgets(pc->processor, MAX_NAME_LEN, stdin);
    trim_string(pc->processor);

    pc->ram_size = get_valid_int("Объем оперативной памяти (ГБ): ", 1);

    printf("Введите производителя компьютера (Dell, HP, Lenovo, Asus и т.д.): ");
    fgets(pc->manufacturer, MAX_NAME_LEN, stdin);
    trim_string(pc->manufacturer);

    pc->is_gaming = get_yes_no("Игровой компьютер?");

    printf("Введите семейство процессоров (например, Core i3, Ryzen 5): ");
    fgets(pc->cpu_family, MAX_FAMILY_LEN, stdin);
    trim_string(pc->cpu_family);

    printf("Введите модель видеокарты (например, NVIDIA GeForce RTX 3060): ");
    fgets(pc->gpu_model, MAX_GPU_MODEL_LEN, stdin);
    trim_string(pc->gpu_model);

    printf("Введите операционную систему (например, Windows 10, Linux): ");
    fgets(pc->os, MAX_OS_LEN, stdin);
    trim_string(pc->os);

    pc->ssd_size = get_valid_int("Объем SSD (ГБ, 0 если нет SSD): ", 0);
    pc->vram_size = get_valid_int("Объем видеопамяти (ГБ): ", 0);
    pc->cpu_clock_speed = get_valid_float("Тактовая частота процессора (ГГц): ", 0.1);
}

void print_computer_table(Computer* computers, int count) {
    if (count == 0) {
        printf("Нет записей для отображения.\n");
        return;
    }

    printf("\n");
    printf("+------------------------+------+-----------------+-----+------------------+----------------------+------------------+------+------+--------+\n");
    printf("| Процессор              | ОЗУ  | Производитель   | Игр.| Семейство CPU    | Видеокарта           | ОС               | SSD  | VRAM | Частота|\n");
    printf("+------------------------+------+-----------------+-----+------------------+----------------------+------------------+------+------+--------+\n");

    for (int i = 0; i < count; i++) {
        // Обрезаем строки до максимальной длины для корректного отображения
        char processor_display[26];
        char manufacturer_display[26];
        char cpu_family_display[26];
        char gpu_display[31];
        char os_display[26];

        strncpy(processor_display, computers[i].processor, 22);
        processor_display[22] = '\0';

        strncpy(manufacturer_display, computers[i].manufacturer, 15);
        manufacturer_display[15] = '\0';

        strncpy(cpu_family_display, computers[i].cpu_family, 16);
        cpu_family_display[16] = '\0';

        strncpy(gpu_display, computers[i].gpu_model, 20);
        gpu_display[20] = '\0';

        strncpy(os_display, computers[i].os, 16);
        os_display[16] = '\0';

        printf("| %-22s | %4d | %-15s |  %d  | %-16s | %-20s | %-16s | %4d | %4d | %6.1f |\n",
            processor_display,
            computers[i].ram_size,
            manufacturer_display,
            computers[i].is_gaming,
            cpu_family_display,
            gpu_display,
            os_display,
            computers[i].ssd_size,
            computers[i].vram_size,
            computers[i].cpu_clock_speed);
    }
    printf("+------------------------+------+-----------------+-----+------------------+----------------------+------------------+------+------+--------+\n");
}

void print_all_computers(Computer* computers, int count) {
    print_computer_table(computers, count);
}

// Поиск в файле по процессору и объему ОЗУ
void search_in_file(void) {
    char filename[FILENAME_LEN];
    printf("Введите имя файла для поиска: ");
    fgets(filename, FILENAME_LEN, stdin);
    trim_string(filename);

    Computer temp_computers[MAX_RECORDS];
    int temp_count = load_from_file(filename, temp_computers, MAX_RECORDS);

    if (temp_count == 0) {
        printf("Файл пуст или не существует!\n");
        return;
    }

    int search_choice;
    printf("\n=== ПОИСК ЗАПИСЕЙ ===\n");
    printf("1. По процессору\n");
    printf("2. По объему оперативной памяти\n");
    printf("3. По обоим полям одновременно\n");
    printf("Выберите тип поиска (1-3): ");
    scanf("%d", &search_choice);
    clear_input_buffer();

    if (search_choice == 1) {
        char processor[MAX_NAME_LEN];
        printf("Введите процессор для поиска: ");
        fgets(processor, MAX_NAME_LEN, stdin);
        trim_string(processor);

        Computer results[MAX_RECORDS];
        int result_count = 0;

        for (int i = 0; i < temp_count; i++) {
            if (strcmp(temp_computers[i].processor, processor) == 0) {
                results[result_count++] = temp_computers[i];
            }
        }

        char message[100];
        snprintf(message, sizeof(message), "=== РЕЗУЛЬТАТЫ ПОИСКА ПО ПРОЦЕССОРУ '%s' ===", processor);
        print_search_results(results, result_count, message);
    }
    else if (search_choice == 2) {
        int ram_size;
        printf("Введите объем оперативной памяти для поиска: ");
        scanf("%d", &ram_size);
        clear_input_buffer();

        Computer results[MAX_RECORDS];
        int result_count = 0;

        for (int i = 0; i < temp_count; i++) {
            if (temp_computers[i].ram_size == ram_size) {
                results[result_count++] = temp_computers[i];
            }
        }

        char message[100];
        snprintf(message, sizeof(message), "=== РЕЗУЛЬТАТЫ ПОИСКА ПО ОБЪЕМУ ОЗУ '%d ГБ' ===", ram_size);
        print_search_results(results, result_count, message);
    }
    else if (search_choice == 3) {
        char processor[MAX_NAME_LEN];
        int ram_size;

        printf("Введите процессор для поиска: ");
        fgets(processor, MAX_NAME_LEN, stdin);
        trim_string(processor);

        printf("Введите объем оперативной памяти для поиска: ");
        scanf("%d", &ram_size);
        clear_input_buffer();

        Computer results[MAX_RECORDS];
        int result_count = 0;

        for (int i = 0; i < temp_count; i++) {
            if (strcmp(temp_computers[i].processor, processor) == 0 &&
                temp_computers[i].ram_size == ram_size) {
                results[result_count++] = temp_computers[i];
            }
        }

        char message[100];
        snprintf(message, sizeof(message), "=== РЕЗУЛЬТАТЫ ПОИСКА ПО ПРОЦЕССОРУ '%s' И ОЗУ '%d ГБ' ===",
            processor, ram_size);
        print_search_results(results, result_count, message);
    }
    else {
        printf("Неверный выбор!\n");
    }
}

void save_to_file(const char* filename, Computer* computers, int count) {
    FILE* file = fopen(filename, "w");
    if (!file) {
        printf("Ошибка открытия файла '%s' для записи!\n", filename);
        return;
    }

    fprintf(file, "Процессор,ОЗУ(ГБ),Производитель,Игровой,Семейство_CPU,Видеокарта,ОС,SSD(ГБ),VRAM(ГБ),Частота(ГГц)\n");

    for (int i = 0; i < count; i++) {
        fprintf(file, "%s,%d,%s,%d,%s,%s,%s,%d,%d,%.1f\n",
            computers[i].processor,
            computers[i].ram_size,
            computers[i].manufacturer,
            computers[i].is_gaming,
            computers[i].cpu_family,
            computers[i].gpu_model,
            computers[i].os,
            computers[i].ssd_size,
            computers[i].vram_size,
            computers[i].cpu_clock_speed);
    }

    fclose(file);
    printf("Данные успешно сохранены в файл '%s' (%d записей)\n", filename, count);
}

int load_from_file(const char* filename, Computer* computers, int max_count) {
    FILE* file = fopen(filename, "r");
    if (!file) {
        printf("Ошибка открытия файла '%s' для чтения!\n", filename);
        return 0;
    }

    char buffer[512];
    fgets(buffer, sizeof(buffer), file);

    int loaded = 0;
    while (loaded < max_count && fgets(buffer, sizeof(buffer), file)) {
        buffer[strcspn(buffer, "\r\n")] = 0;

        char* token = strtok(buffer, ",");
        if (!token) continue;

        strncpy(computers[loaded].processor, token, MAX_NAME_LEN - 1);
        computers[loaded].processor[MAX_NAME_LEN - 1] = '\0';

        token = strtok(NULL, ",");
        if (token) computers[loaded].ram_size = atoi(token);

        token = strtok(NULL, ",");
        if (token) {
            strncpy(computers[loaded].manufacturer, token, MAX_NAME_LEN - 1);
            computers[loaded].manufacturer[MAX_NAME_LEN - 1] = '\0';
        }

        token = strtok(NULL, ",");
        if (token) computers[loaded].is_gaming = atoi(token);

        token = strtok(NULL, ",");
        if (token) {
            strncpy(computers[loaded].cpu_family, token, MAX_FAMILY_LEN - 1);
            computers[loaded].cpu_family[MAX_FAMILY_LEN - 1] = '\0';
        }

        token = strtok(NULL, ",");
        if (token) {
            strncpy(computers[loaded].gpu_model, token, MAX_GPU_MODEL_LEN - 1);
            computers[loaded].gpu_model[MAX_GPU_MODEL_LEN - 1] = '\0';
        }

        token = strtok(NULL, ",");
        if (token) {
            strncpy(computers[loaded].os, token, MAX_OS_LEN - 1);
            computers[loaded].os[MAX_OS_LEN - 1] = '\0';
        }

        token = strtok(NULL, ",");
        if (token) computers[loaded].ssd_size = atoi(token);

        token = strtok(NULL, ",");
        if (token) computers[loaded].vram_size = atoi(token);

        token = strtok(NULL, ",\n");
        if (token) computers[loaded].cpu_clock_speed = atof(token);

        loaded++;
    }

    fclose(file);
    return loaded;
}

// Функции сравнения для сортировки
int compare_processor(const void* a, const void* b) {
    return strcmp(((Computer*)a)->processor, ((Computer*)b)->processor);
}

int compare_gaming(const void* a, const void* b) {
    return ((Computer*)a)->is_gaming - ((Computer*)b)->is_gaming;
}

// Сортировка из файла
void sort_and_print_from_file(void) {
    char filename[FILENAME_LEN];
    printf("Введите имя файла для сортировки: ");
    fgets(filename, FILENAME_LEN, stdin);
    trim_string(filename);

    Computer temp_computers[MAX_RECORDS];
    int temp_count = load_from_file(filename, temp_computers, MAX_RECORDS);

    if (temp_count == 0) {
        printf("Файл пуст или не существует!\n");
        return;
    }

    int sort_choice;
    printf("\n=== СОРТИРОВКА ЗАПИСЕЙ ===\n");
    printf("1. По процессору (алфавитный порядок)\n");
    printf("2. По игровому статусу (сначала неигровые, потом игровые)\n");
    printf("Выберите тип сортировки (1-2): ");
    scanf("%d", &sort_choice);
    clear_input_buffer();

    if (sort_choice == 1) {
        qsort(temp_computers, temp_count, sizeof(Computer), compare_processor);
        printf("\n=== ОТСОРТИРОВАННЫЕ ЗАПИСИ (ПО ПРОЦЕССОРУ) ===\n");
        print_all_computers(temp_computers, temp_count);
    }
    else if (sort_choice == 2) {
        qsort(temp_computers, temp_count, sizeof(Computer), compare_gaming);
        printf("\n=== ОТСОРТИРОВАННЫЕ ЗАПИСИ (ПО ИГРОВОМУ СТАТУСУ) ===\n");
        print_all_computers(temp_computers, temp_count);
    }
    else {
        printf("Неверный выбор!\n");
    }
}

void modify_record_in_file(void) {
    char filename[FILENAME_LEN];
    printf("Введите имя файла: ");
    fgets(filename, FILENAME_LEN, stdin);
    trim_string(filename);

    Computer temp_computers[MAX_RECORDS];
    int temp_count = load_from_file(filename, temp_computers, MAX_RECORDS);

    if (temp_count == 0) {
        printf("Файл пуст или не существует!\n");
        return;
    }

    printf("\n=== Содержимое файла ===\n");
    print_all_computers(temp_computers, temp_count);

    int index;
    printf("Введите индекс записи для изменения (0-%d): ", temp_count - 1);
    scanf("%d", &index);
    clear_input_buffer();

    if (index >= 0 && index < temp_count) {
        printf("\n=== ИЗМЕНЕНИЕ ЗАПИСИ %d ===\n", index);
        printf("Текущие данные:\n");

        printf("+------------------------+------+-----------------+-----+------------------+----------------------+------------------+------+------+--------+\n");
        printf("| Процессор              | ОЗУ  | Производитель   | Игр.| Семейство CPU    | Видеокарта           | ОС               | SSD  | VRAM | Частота|\n");
        printf("+------------------------+------+-----------------+-----+------------------+----------------------+------------------+------+------+--------+\n");

        // Обрезаем строки для отображения
        char processor_display[26];
        char manufacturer_display[26];
        char cpu_family_display[26];
        char gpu_display[31];
        char os_display[26];

        strncpy(processor_display, temp_computers[index].processor, 22);
        processor_display[22] = '\0';

        strncpy(manufacturer_display, temp_computers[index].manufacturer, 15);
        manufacturer_display[15] = '\0';

        strncpy(cpu_family_display, temp_computers[index].cpu_family, 16);
        cpu_family_display[16] = '\0';

        strncpy(gpu_display, temp_computers[index].gpu_model, 20);
        gpu_display[20] = '\0';

        strncpy(os_display, temp_computers[index].os, 16);
        os_display[16] = '\0';

        printf("| %-22s | %4d | %-15s |  %d  | %-16s | %-20s | %-16s | %4d | %4d | %6.1f |\n",
            processor_display,
            temp_computers[index].ram_size,
            manufacturer_display,
            temp_computers[index].is_gaming,
            cpu_family_display,
            gpu_display,
            os_display,
            temp_computers[index].ssd_size,
            temp_computers[index].vram_size,
            temp_computers[index].cpu_clock_speed);
        printf("+------------------------+------+-----------------+-----+------------------+----------------------+------------------+------+------+--------+\n");

        printf("\nВведите новые данные:\n");
        fill_computer_data(&temp_computers[index]);

        save_to_file(filename, temp_computers, temp_count);
        printf("Запись успешно изменена и сохранена в файл!\n");
    }
    else {
        printf("Неверный индекс!\n");
    }
}

void add_records_to_file(void) {
    char filename[FILENAME_LEN];
    printf("Введите имя файла: ");
    fgets(filename, FILENAME_LEN, stdin);
    trim_string(filename);

    Computer new_computers[MAX_RECORDS];
    int new_count;

    printf("Сколько новых записей добавить? ");
    scanf("%d", &new_count);
    clear_input_buffer();

    if (new_count <= 0) {
        printf("Некорректное количество записей!\n");
        return;
    }

    for (int i = 0; i < new_count; i++) {
        printf("\n=== Новая запись %d из %d ===\n", i + 1, new_count);
        fill_computer_data(&new_computers[i]);
    }

    Computer existing_computers[MAX_RECORDS];
    int existing_count = load_from_file(filename, existing_computers, MAX_RECORDS);

    int total_count = existing_count;
    for (int i = 0; i < new_count && total_count < MAX_RECORDS; i++) {
        existing_computers[total_count] = new_computers[i];
        total_count++;
    }

    save_to_file(filename, existing_computers, total_count);

    printf("Успешно добавлено %d новых записей в файл '%s'\n", new_count, filename);
}
