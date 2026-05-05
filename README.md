# seq
#include <stdio.h>
#include <time.h>
#include <math.h>

#define LIMIT 100000000

// ฟังก์ชันตรวจสอบจำนวนเฉพาะ
int isprime(long long n) {
    if (n < 2) return 0;
    if (n == 2) return 1;
    if (n % 2 == 0) return 0;
    
    long long sqrt_n = (long long)sqrt((double)n);
    for (long long i = 3; i <= sqrt_n; i += 2) {
        if (n % i == 0) return 0;
    }
    return 1;
}

int main(int argc, char **argv)
{
    long long i;
    int count = 4;  // 2,3,5,7
    time_t start_time, end_time;
    
    start_time = time(NULL);
    
    for (i = 11; i <= LIMIT; i += 2) {
        if (isprime(i)) count++;
    }
    
    end_time = time(NULL);
    printf("Numbers of prime <= %d is %d (Elapsed time %.2lf sec)\n",
           LIMIT, count, difftime(end_time, start_time));
    
    return 0;
}



โค้ด Parallel

#include <stdio.h>
#include <math.h>
#include "mpi.h"

#define LIMIT 100000000

// ฟังก์ชันตรวจสอบจำนวนเฉพาะ
int isprime(long long n) {
    if (n < 2) return 0;
    if (n == 2) return 1;
    if (n % 2 == 0) return 0;
    
    long long sqrt_n = (long long)sqrt((double)n);
    for (long long i = 3; i <= sqrt_n; i += 2) {
        if (n % i == 0) return 0;
    }
    return 1;
}

int main(int argc, char **argv)
{
    int rank, ntasks;
    int local_count = 0;
    int total_count = 4;  // 2,3,5,7
    long long i;
    double start_time, end_time;
    
    MPI_Init(&argc, &argv);
    MPI_Comm_size(MPI_COMM_WORLD, &ntasks);
    MPI_Comm_rank(MPI_COMM_WORLD, &rank);
    
    start_time = MPI_Wtime();
    
    // แต่ละ rank ตรวจสอบส่วนของตัวเอง
    for (i = 11 + rank * 2; i <= LIMIT; i += ntasks * 2) {
        if (isprime(i)) {
            local_count++;
        }
    }
    
    // รวมผลจากทุก rank
    MPI_Reduce(&local_count, &total_count, 1, MPI_INT, MPI_SUM, 0, MPI_COMM_WORLD);
    
    end_time = MPI_Wtime();
    
    // เฉพาะ Rank 0 แสดงผล
    if (rank == 0) {
        total_count += 4;  // บวก 2,3,5,7
        printf("Numbers of prime <= %d is %d (Elapsed time %.2lf sec)\n",
               LIMIT, total_count, (end_time - start_time));
    }
    
    MPI_Finalize();
    return 0;
}



