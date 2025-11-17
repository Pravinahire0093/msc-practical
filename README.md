Assignment 1 
 
Q.1) Take multiple files as Command Line Arguments and print their inode 
numbers and file types [10 Marks ] 
 
#include <stdio.h> 
#include <sys/stat.h> 
#include <stdlib.h> 
int main(int argc, char *argv[]) { 
    struct stat fileStat; 
    int i; 
    if (argc < 2) { 
        printf("Usage: %s <file1> <file2> ...\n", argv[0]); 
        exit(1); 
    } 
    for (i = 1; i < argc; i++) { 
        if (stat(argv[i], &fileStat) == -1) { 
            perror(argv[i]); 
            continue; 
        } 
        printf("File: %s\n", argv[i]); 
        printf("Inode Number: %ld\n", fileStat.st_ino); 
        if (S_ISREG(fileStat.st_mode)) 
            printf("Type: Regular File\n"); 
        else if (S_ISDIR(fileStat.st_mode)) 
            printf("Type: Directory\n"); 
        else if (S_ISCHR(fileStat.st_mode)) 
            printf("Type: Character Device\n"); 
        else if (S_ISBLK(fileStat.st_mode)) 
            printf("Type: Block Device\n"); 
        else if (S_ISFIFO(fileStat.st_mode)) 
            printf("Type: FIFO (Pipe)\n"); 
        else if (S_ISLNK(fileStat.st_mode)) 
            printf("Type: Symbolic Link\n"); 
        else if (S_ISSOCK(fileStat.st_mode)) 
            printf("Type: Socket\n"); 
        else 
            printf("Type: Unknown\n"); 
        printf("---------------------------------\n"); 
    } 
    return 0; 
} 
OUTPUT:- 
File: file_info.c 
Inode Number: 4062587 
Type: Regular File --------------------------------- 
File: /etc 
Inode Number: 2 
Type: Directory --------------------------------- 
File: passwd 
Inode Number: 123456 
Type: Regular File --------------------------------- 
 
 
 Q.2) Write a C program to send SIGALRM signal by child process to parent process and 
parent process make a provision to catch the signal and display alarm is fired.(Use 
Kill, fork, signal and sleep system call) [20 Marks ] 
 
#include <stdio.h> 
#include <signal.h> 
#include <unistd.h> 
#include <sys/types.h> 
void handle_alarm(int sig) { 
    printf("Alarm is fired! (Caught signal %d)\n", sig); 
} 
int main() { 
    pid_t pid; 
    signal(SIGALRM, handle_alarm);  // parent sets up signal handler 
    pid = fork(); 
    if (pid < 0) { 
        perror("Fork failed"); 
        return 1; 
    } else if (pid == 0) { 
        // Child process 
        sleep(2); // give parent time to set handler 
        printf("Child sending SIGALRM to parent...\n"); 
        kill(getppid(), SIGALRM); 
    } else { 
        // Parent process 
printf("Parent waiting for alarm...\n"); 
pause(); // wait for signal 
printf("Parent received alarm and continues...\n"); 
} 
return 0; 
} 
OUTPUT:- 
Parent waiting for alarm... 
Child sending SIGALRM to parent... 
Alarm is fired! (Caught signal 14) 
Parent received alarm and continues... 
SUBJECT: CS-504-MJP: Lab Course on CS-501-MJ 
(Advanced Operating System) 
Assignment 2 
Q.1) Write a C program to find file properties such as inode number, number of hard 
link, File permissions, File size, File access and modification time and so on of a given 
f
 ile using stat() system call. [10 Marks ] 
#include <stdio.h> 
#include <stdlib.h> 
#include <sys/stat.h> 
#include <unistd.h> 
#include <pwd.h> 
#include <grp.h> 
#include <time.h> 
void print_permissions(mode_t mode) { 
printf("File permissions: "); 
printf( (S_ISDIR(mode)) ? "d" : "-"); 
printf( (mode & S_IRUSR) ? "r" : "-"); 
printf( (mode & S_IWUSR) ? "w" : "-"); 
printf( (mode & S_IXUSR) ? "x" : "-"); 
printf( (mode & S_IRGRP) ? "r" : "-"); 
printf( (mode & S_IWGRP) ? "w" : "-"); 
printf( (mode & S_IXGRP) ? "x" : "-"); 
printf( (mode & S_IROTH) ? "r" : "-"); 
printf( (mode & S_IWOTH) ? "w" : "-"); 
printf( (mode & S_IXOTH) ? "x" : "-"); 
printf("\n"); 
} 
 
int main(int argc, char *argv[]) { 
    if(argc != 2){ 
        printf("Usage: %s <filename>\n", argv[0]); 
        return 1; 
    } 
    struct stat fileStat; 
    if(stat(argv[1], &fileStat) < 0){ 
        perror("stat"); 
        return 1; 
    } 
 
    printf("File: %s\n", argv[1]); 
    printf("Inode number: %lu\n", (unsigned long)fileStat.st_ino); 
    printf("Number of hard links: %lu\n", (unsigned long)fileStat.st_nlink); 
    print_permissions(fileStat.st_mode); 
    printf("File size: %ld bytes\n", (long)fileStat.st_size); 
 
    struct passwd *pw = getpwuid(fileStat.st_uid); 
    struct group  *gr = getgrgid(fileStat.st_gid); 
    printf("Owner UID: %u (%s)\n", fileStat.st_uid, pw ? pw->pw_name : "Unknown"); 
    printf("Group GID: %u (%s)\n", fileStat.st_gid, gr ? gr->gr_name : "Unknown"); 
     
    printf("Last access  : %s", ctime(&fileStat.st_atime)); 
    printf("Last modified: %s", ctime(&fileStat.st_mtime)); 
    printf("Last status change: %s", ctime(&fileStat.st_ctime)); 
    return 0; 
} 
OutPut: 
File: test.txt 
Inode number: 2345678 
Number of hard links: 1 
File permissions: -rw-r--r-- 
File size: 42 bytes 
Owner UID: 1000 (yourusername) 
Group GID: 1000 (yourgroup) 
Last access  : Sat Nov  8 17:05:22 2025 
Last modified: Sat Nov  8 16:59:12 2025 
Last status change: Sat Nov  8 16:59:12 2025 
Q.2) Write a C program that catches the ctrl-c (SIGINT) signal for the first time and 
display the appropriate message and exits on pressing ctrl-c again. [20 Marks ] 
#include <stdio.h> 
#include <signal.h> 
#include <stdlib.h> 
volatile sig_atomic_t sigint_count = 0; 
void handle_sigint(int sig) { 
sigint_count++; 
if(sigint_count == 1){ 
printf("\nCaught SIGINT (Ctrl-C) for the first time. Press Ctrl-C again to exit.\n"); 
} else { 
printf("\nSecond SIGINT received. Exiting...\n"); 
exit(0); 
} 
} 
int main() { 
struct sigaction sa; 
sa.sa_handler = handle_sigint; 
sigemptyset(&sa.sa_mask); 
sa.sa_flags = 0; 
sigaction(SIGINT, &sa, NULL); 
printf("Press Ctrl-C. The first time displays a message. The second time exits.\n"); 
while(1) { 
pause();  // Wait for signals 
} 
return 0; 
} 
OutPut: 
Press Ctrl-C. The first time displays a message. The second time exits. 
Caught SIGINT (Ctrl-C) for the first time. Press Ctrl-C again to exit. 
Second SIGINT received. Exiting... 
SUBJECT: CS-504-MJP: Lab Course on CS-501-MJ 
(Advanced Operating System) 
Assignment 3 
Q.1) Print the type of file and inode number where file name accepted through 
Command Line [10 Marks ] 
#include <stdio.h> 
#include <sys/stat.h> 
const char* file_type(mode_t mode) { 
if (S_ISREG(mode))  return "Regular File"; 
if (S_ISDIR(mode))  return "Directory"; 
if (S_ISCHR(mode))  return "Character Device"; 
if (S_ISBLK(mode))  return "Block Device"; 
if (S_ISFIFO(mode)) return "FIFO/Pipe"; 
if (S_ISLNK(mode))  return "Symbolic Link"; 
if (S_ISSOCK(mode)) return "Socket"; 
return "Unknown"; 
} 
int main(int argc, char *argv[]) { 
if (argc != 2) { 
printf("Usage: %s <filename>\n", argv[0]); 
return 1; 
} 
struct stat st; 
if (stat(argv[1], &st) < 0) { 
perror("stat"); 
return 1; 
} 
printf("File: %s\n", argv[1]); 
printf("File type: %s\n", file_type(st.st_mode)); 
printf("Inode number: %lu\n", (unsigned long)st.st_ino); 
return 0; 
} 
OutPut : 
File: test.txt 
File type: Regular File 
Inode number: 2345678 
If you run with a directory instead (e.g., /tmp): 
File: /tmp 
File type: Directory 
Inode number: 1024 
Q.2) Write a C program which creates a child process to run linux/ unix command or 
any user defined program. The parent process set the signal handler for death of child 
signal and Alarm signal. If a child process does not complete its execution in 5 second 
then parent process kills child process. [20 Marks ] 
#include <stdio.h> 
#include <stdlib.h> 
#include <unistd.h> 
#include <signal.h> 
#include <sys/wait.h> 
pid_t child_pid = -1; // Track child PID 
void sigchld_handler(int sig) { 
printf("Child process terminated (SIGCHLD received).\n"); 
// Optionally reap the child process to avoid zombie 
waitpid(child_pid, NULL, WNOHANG); 
} 
void sigalrm_handler(int sig) { 
printf("Alarm Fired: Child process exceeded time limit. Killing child...\n"); 
if (child_pid > 0) { 
kill(child_pid, SIGKILL); 
} 
} 
int main(int argc, char *argv[]) { 
if (argc < 2) { 
        printf("Usage: %s <command> [arg1 ... argN]\n", argv[0]); 
        exit(1); 
    } 
    // Set up signal handlers for SIGCHLD (child exit) and SIGALRM (timeout) 
    signal(SIGCHLD, sigchld_handler); 
    signal(SIGALRM, sigalrm_handler); 
 
    child_pid = fork(); 
 
    if (child_pid < 0) { 
        perror("fork"); 
        exit(1); 
    } 
    if (child_pid == 0) { 
        // Child executes the given program/command 
        execvp(argv[1], &argv[1]); 
        // If execvp fails 
        perror("execvp"); 
        exit(1); 
    } else { 
        // Parent process 
        alarm(5); // Set a 5 seconds timer 
        int status; 
        pid_t ret; 
        ret = waitpid(child_pid, &status, 0); // Wait for child to exit 
        alarm(0); // Cancel alarm if child exited in time 
if (ret == child_pid) { 
printf("Child exited normally.\n"); 
} 
} 
return 0; 
} 
OutPut: 
./a.out sleep 10 
Alarm Fired: Child process exceeded time limit. Killing child... 
Child process terminated (SIGCHLD received). 
Child process terminated (SIGCHLD received). 
Child exited normally. 
./a.out sleep 2 
Child process terminated (SIGCHLD received). 
Child exited normally. 
Usage: ./a.out <command> [arg1 ... argN]  
SUBJECT: CS-504-MJP: Lab Course on CS-501-MJ 
(Advanced Operating System) 
Assignment 4 
 
Q.1) Write a C program to find whether a given files passed through command line 
arguments are present in current directory or not. [10 Marks ] 
 
#include <stdio.h> 
#include <sys/stat.h> 
 
int main(int argc, char *argv[]) { 
    if (argc < 2) { 
        printf("Usage: %s <file1> [file2 ... fileN]\n", argv[0]); 
        return 1; 
    } 
    for (int i = 1; i < argc; ++i) { 
        struct stat st; 
        if (stat(argv[i], &st) == 0) { 
            printf("File '%s' is present in current directory.\n", argv[i]); 
        } else { 
            printf("File '%s' is NOT present in current directory.\n", argv[i]); 
        } 
    } 
    return 0; 
} 
OUTPUT: 
./a.out notes.txt main.c missing.txt report.pdf 
File 'notes.txt' is present in current directory. 
File 'main.c' is present in current directory. 
File 'missing.txt' is NOT present in current directory. 
File 'report.pdf' is present in current directory. 
Q.2) Write a C program which creates a child process and child process catches a 
signal SIGHUP, SIGINT and SIGQUIT. The Parent process send a SIGHUP or SIGINT 
signal after every 3 seconds, at the end of 15 second parent send SIGQUIT signal to 
child and child terminates by displaying message "My Papa has Killed me!!!”. [20 
Marks ] 
#include <stdio.h> 
#include <stdlib.h> 
#include <unistd.h> 
#include <signal.h> 
volatile sig_atomic_t running = 1; 
// Signal handlers for the child 
void handle_sighup(int sig) { 
printf("Child: Received SIGHUP\n"); 
} 
void handle_sigint(int sig) { 
printf("Child: Received SIGINT\n"); 
} 
void handle_sigquit(int sig) { 
    printf("My Papa has Killed me!!!\n"); 
    running = 0; 
} 
 
int main() { 
    pid_t pid = fork(); 
 
    if (pid < 0) { 
        perror("fork"); 
        exit(1); 
    } 
    if (pid == 0) { 
        // Child process: install signal handlers 
        signal(SIGHUP, handle_sighup); 
        signal(SIGINT, handle_sigint); 
        signal(SIGQUIT, handle_sigquit); 
 
        while (running) { 
            pause(); // Wait for signals 
        } 
        exit(0); 
    } else { 
        // Parent process 
        int elapsed = 0; 
        while (elapsed < 15) { 
            sleep(3); 
            elapsed += 3; 
            if (elapsed < 15) { 
                // Alternate between SIGHUP and SIGINT 
                if (elapsed % 6 == 0) 
                    kill(pid, SIGHUP); 
                else 
                    kill(pid, SIGINT); 
            } 
        } 
        // After 15 seconds, send SIGQUIT 
        kill(pid, SIGQUIT); 
        wait(NULL); // Optionally wait for child to exit 
    } 
    return 0; 
} 
OUTPUT: 
Child: Received SIGINT 
Child: Received SIGHUP 
Child: Received SIGINT 
Child: Received SIGHUP 
Child: Received SIGINT 
My Papa has Killed me!!!  
SUBJECT: CS-504-MJP: Lab Course on CS-501-MJ 
(Advanced Operating System) 
Assignment 5 
 
Q.1) Read the current directory and display the name of the files, no of files in current 
directory      [10 Marks ] 
 
#include <stdio.h> 
#include <dirent.h> 
 
int main() { 
    DIR *dir; 
    struct dirent *entry; 
    int count = 0; 
 
    dir = opendir("."); 
    if (dir == NULL) { 
        perror("Unable to open directory"); 
        return 1; 
    } 
 
    printf("Files in current directory:\n"); 
    while ((entry = readdir(dir)) != NULL) { 
        // Skip "." and ".." 
        if (entry->d_name[0] == '.' &&  
            (entry->d_name[1] == '\0' ||  
             (entry->d_name[1] == '.' && entry->d_name[2] == '\0'))) { 
            continue; 
} 
printf("%s\n", entry->d_name); 
count++; 
} 
closedir(dir); 
printf("Total number of files: %d\n", count); 
return 0; 
} 
OUTPUT: 
Files in current directory: 
main.c 
test.txt 
data.csv 
output.txt 
Total number of files: 4 
Q.2) Write a C program to create an unnamed pipe. The child process will write 
following three messages to pipe and parent process display it. 
Message1 = “Hello World”  
Message2 = “Hello SPPU”  
Message3 = “Linux is Funny”                                                                
#include <stdio.h> 
#include <unistd.h> 
#include <stdlib.h> 
#include <string.h> 
[20 Marks] 
 
int main() { 
    int fd[2]; 
    pid_t pid; 
    char buffer[100]; 
 
    if (pipe(fd) == -1) { 
        perror("pipe"); 
        exit(1); 
    } 
    pid = fork(); 
    if (pid < 0) { 
        perror("fork"); 
        exit(1); 
    } 
    if (pid == 0) { 
        // Child process -- writer 
        close(fd[0]); // Close read-end 
        char *messages[] = { 
            "Hello World", 
            "Hello SPPU", 
            "Linux is Funny" 
        }; 
        for(int i = 0; i < 3; i++) { 
            write(fd[1], messages[i], strlen(messages[i]) + 1); // +1 for '\0' 
        } 
        close(fd[1]); 
        exit(0); 
    } else { 
        // Parent process -- reader 
        close(fd[1]); // Close write-end 
        printf("Messages from child:\n"); 
        for(int i = 0; i < 3; i++) { 
            read(fd[0], buffer, sizeof(buffer)); 
            printf("%s\n", buffer); 
        } 
        close(fd[0]); 
    } 
    return 0; 
} 
OUTPUT: 
Messages from child: 
Hello World 
Hello SPPU 
Linux is Funny 
