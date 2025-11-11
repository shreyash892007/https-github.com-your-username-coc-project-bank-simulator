 /*Project Description
A C program that simulates a realistic customer queue in a bank over an 8-hour day (480 minutes).
 The program models customer arrivals using a Poisson distribution, manages a dynamic queue of
 customers with linked lists (pointers, structs, malloc/free), and simulates teller service times.
The simulation outputs a report including average, median, mode, standard deviation, and maximum wait times of customers.

Concepts Used
C Concepts:
For loops
If/else logic
Functions (customer_arrives(), serve_customer())
Structs
Pointers
Dynamic memory allocation: malloc(), free()
Math Concepts:
Poisson Distribution (for customer arrivals)
Central tendencies: Mean, Median, Mode
Standard deviation
How to Compile
gcc coc-project-bank-simulator.c

How to Run
After compiling, run: "./a.exe"
*/
#include <stdio.h>
#include <stdlib.h>
#include <math.h>
#include <time.h>

#define SIMULATION_MINUTES 480
#define MIN_SERVICE_TIME 2
#define MAX_SERVICE_TIME 3

typedef struct Customer
{
    int arrival_minute;
    struct Customer *next;
} Customer;

typedef struct
{
    int busy_until;
    Customer *current_customer;
} Teller;

// Function prototypes
int poisson(double lambda);
void enqueue(Customer **head, Customer **tail, int arrival_minute);
Customer *dequeue(Customer **head, Customer **tail);
int compare_ints(const void *a, const void *b);
int mode(int *sorted, int count);
void calculate_statistics(int *wait_times, int count);

int main()
{
    srand(time(NULL));

    double lambda;
    printf("Enter average customers per minute (lambda): ");
    scanf("%lf", &lambda);

    Customer *queue_head = NULL;
    Customer *queue_tail = NULL;
    Teller teller = {.busy_until = 0, .current_customer = NULL};

    int capacity = 100;
    int *wait_times = malloc(capacity * sizeof(int));
    int wait_count = 0;

    for (int minute = 0; minute < SIMULATION_MINUTES; minute++)
    {
        int arrivals = poisson(lambda);
        for (int i = 0; i < arrivals; i++)
        {
            enqueue(&queue_head, &queue_tail, minute);
        }

        if (teller.current_customer && minute >= teller.busy_until)
        {
            int wait_time = teller.busy_until - teller.current_customer->arrival_minute - (teller.busy_until - minute);
            if (wait_time < 0)
                wait_time = 0;

            if (wait_count == capacity)
            {
                capacity *= 2;
                wait_times = realloc(wait_times, capacity * sizeof(int));
            }
            wait_times[wait_count++] = wait_time;
            free(teller.current_customer);
            teller.current_customer = NULL;
        }

        if (!teller.current_customer && queue_head != NULL)
        {
            teller.current_customer = dequeue(&queue_head, &queue_tail);
            int service_time = MIN_SERVICE_TIME + rand() % (MAX_SERVICE_TIME - MIN_SERVICE_TIME + 1);
            teller.busy_until = minute + service_time;
        }
    }

    int minute = SIMULATION_MINUTES;
    while (teller.current_customer || queue_head)
    {
        if (teller.current_customer && minute >= teller.busy_until)
        {
            int wait_time = teller.busy_until - teller.current_customer->arrival_minute - (teller.busy_until - minute);
            if (wait_time < 0)
                wait_time = 0;

            if (wait_count == capacity)
            {
                capacity *= 2;
                wait_times = realloc(wait_times, capacity * sizeof(int));
            }
            wait_times[wait_count++] = wait_time;
            free(teller.current_customer);
            teller.current_customer = NULL;
        }

        if (!teller.current_customer && queue_head != NULL)
        {
            teller.current_customer = dequeue(&queue_head, &queue_tail);
            int service_time = MIN_SERVICE_TIME + rand() % (MAX_SERVICE_TIME - MIN_SERVICE_TIME + 1);
            teller.busy_until = minute + service_time;
        }
        minute++;
    }

    if (wait_count == 0)
    {
        printf("No customers served during the day.\n");
    }
    else
    {
        calculate_statistics(wait_times, wait_count);
    }

    free(wait_times);
    return 0;
}

int poisson(double lambda)
{
    int k = 0;
    double p = 1.0;
    double L = exp(-lambda);
    do
    {
        k++;
        p *= ((double)rand() / RAND_MAX);
    } while (p > L);
    return k - 1;
}

void enqueue(Customer **head, Customer **tail, int arrival_minute)
{
    Customer *new_customer = malloc(sizeof(Customer));
    new_customer->arrival_minute = arrival_minute;
    new_customer->next = NULL;
    if (*tail == NULL)
    {
        *head = new_customer;
        *tail = new_customer;
    }
    else
    {
        (*tail)->next = new_customer;
        *tail = new_customer;
    }
}

Customer *dequeue(Customer **head, Customer **tail)
{
    if (*head == NULL)
        return NULL;
    Customer *front = *head;
    *head = front->next;
    if (*head == NULL)
        *tail = NULL;
    front->next = NULL;
    return front;
}

int compare_ints(const void *a, const void *b)
{
    return (*(int *)a - *(int *)b);
}

int mode(int *sorted, int count)
{
    int mode = sorted[0], mode_count = 1, current_count = 1;
    for (int i = 1; i < count; i++)
    {
        if (sorted[i] == sorted[i - 1])
            current_count++;
        else
        {
            if (current_count > mode_count)
            {
                mode = sorted[i - 1];
                mode_count = current_count;
            }
            current_count = 1;
        }
    }
    if (current_count > mode_count)
        mode = sorted[count - 1];
    return mode;
}

void calculate_statistics(int *wait_times, int count)
{
    qsort(wait_times, count, sizeof(int), compare_ints);

    double sum = 0;
    for (int i = 0; i < count; i++)
        sum += wait_times[i];
    double mean = sum / count;

    double median;
    if (count % 2 == 0)
        median = (wait_times[count / 2 - 1] + wait_times[count / 2]) / 2.0;
    else
        median = wait_times[count / 2];

    int m = mode(wait_times, count);

    double sq_sum = 0;
    for (int i = 0; i < count; i++)
        sq_sum += (wait_times[i] - mean) * (wait_times[i] - mean);
    double stddev = sqrt(sq_sum / count);

    int max = wait_times[count - 1];

    printf("\n--- Simulation Report ---\n");
    printf("Customers served: %d\n", count);
    printf("Mean wait time: %.2f minutes\n", mean);
    printf("Median wait time: %.2f minutes\n", median);
    printf("Mode wait time: %d minutes\n", m);
    printf("Standard deviation: %.2f minutes\n", stddev);
    printf("Maximum wait time: %d minutes\n", max);
}

