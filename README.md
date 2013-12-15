Warplife
========

http://www.kuro5hin.org/story/2013/12/15/05559/463


[+] [P]
Super Secret Warp Life source dump

By cockskin horsesuit [Watch this Diary] in cockskin horsesuit's Diary
Sun Dec 15, 2013 at 12:55:59 AM EST 
Tags: sodomy, horsecock, G4C, Warp Life (all tags)	
	
In honor of the Global Day of Coderetreat, I am posting the full source code to the beta version of Warp Life below.


	Alter M and N for a different grid size.
Important note: Requires roughly 9*M*N file descriptors (ulimit -n) and M*N processes (ulimit -u).  This will probably require adjusting your ulimit and/or the limits in /etc/security/limits.conf and/or running as root (pick any 2).  Or you can shrink the board down really small to use less resources.
Crawford is working hard on improvements to the algorithm to consume less electricity.

To compile: cc -o warplife warplife.c

warplife.c:


#include <stdio.h>
#include <stdlib.h>
#include <fcntl.h>
#include <sys/types.h>
#include <sys/wait.h>
#include <time.h>
#include <assert.h>

enum {nw, n, ne, w, e, sw, s, se, display};

#define M 78
#define N 24

int devnull;
int devzero;
int pipes[M][N][9][2];

void setboundary(int pipes[2])
{
  pipes[0] = devzero;
  pipes[1] = devnull;
}

void
tick(char start, int pipes[9][2])
{
  char alive = start;

  while (1)
    {
      int i, neighbors=0;
      for (i=0;i<9;i++)
    {
      write(pipes[i][1], &alive, 1);
    }
      for (i=0;i<8;i++)
    {
      int b;
      char c;
      do {
        b = read(pipes[i][0], &c, 1);
      } while (b != 1);
      neighbors += c;
    }
      alive = neighbors == 3 || (neighbors == 2 && alive);
    }
}

void initpipes()
{
  int i, j;
  devnull = open("/dev/null", O_WRONLY);
  devzero = open("/dev/zero", O_RDONLY);
  for (i=0;i<M;i++)
    for (j=0;j<N;j++)
      {
    assert(0 == pipe(pipes[i][j][display]));
    if (i == 0)
      {
        setboundary(pipes[i][j][nw]);
        setboundary(pipes[i][j][w]);
        setboundary(pipes[i][j][sw]);
      }
    else
      {
        int pipefd1[2];
        int pipefd2[2];
        if (j != 0)
          {
        assert(0 == pipe(pipefd1));
        assert(0 == pipe(pipefd2));
        pipes[i][j][nw][0] = pipefd1[0];
        pipes[i][j][nw][1] = pipefd2[1];
        pipes[i-1][j-1][se][0] = pipefd2[0];
        pipes[i-1][j-1][se][1] = pipefd1[1];
          }
        else
          {
        setboundary(pipes[i][j][nw]);
          }
        assert(0 == pipe(pipefd1));
        assert(0 == pipe(pipefd2));
        pipes[i][j][w][0] = pipefd1[0];
        pipes[i][j][w][1] = pipefd2[1];
        pipes[i-1][j][e][0] = pipefd2[0];
        pipes[i-1][j][e][1] = pipefd1[1];
        if (j != N-1)
          {
        assert(0 == pipe(pipefd1));
        assert(0 == pipe(pipefd2));
        pipes[i][j][sw][0] = pipefd1[0];
        pipes[i][j][sw][1] = pipefd2[1];
        pipes[i-1][j+1][ne][0] = pipefd2[0];
        pipes[i-1][j+1][ne][1] = pipefd1[1];
          }
        else
          {
        setboundary(pipes[i][j][sw]);
          }
      }
    if (j == 0)
      {
        setboundary(pipes[i][j][n]);
        setboundary(pipes[i][j][ne]);
      }
    else
      {
        int pipefd1[2];
        int pipefd2[2];
        assert(0 == pipe(pipefd1));
        assert(0 == pipe(pipefd2));
        pipes[i][j][n][0] = pipefd1[0];
        pipes[i][j][n][1] = pipefd2[1];
        pipes[i][j-1][s][0] = pipefd2[0];
        pipes[i][j-1][s][1] = pipefd1[1];
        if (i != M-1)
          {
        assert(0 == pipe(pipefd1));
        assert(0 == pipe(pipefd2));
        pipes[i][j][ne][0] = pipefd1[0];
        pipes[i][j][ne][1] = pipefd2[1];
        pipes[i+1][j-1][sw][0] = pipefd2[0];
        pipes[i+1][j-1][sw][1] = pipefd1[1];
          }
        else
          {
        setboundary(pipes[i][j][ne]);
          }        
      }
    if (i == M-1)
      {
        setboundary(pipes[i][j][e]);
        setboundary(pipes[i][j][se]);
      }
    else
      {
        int pipefd1[2];
        int pipefd2[2];
        assert(0 == pipe(pipefd1));
        assert(0 == pipe(pipefd2));
        pipes[i][j][e][0] = pipefd1[0];
        pipes[i][j][e][1] = pipefd2[1];
        pipes[i+1][j][w][0] = pipefd2[0];
        pipes[i+1][j][w][1] = pipefd1[1];
        if (j != N-1)
          {
        assert(0 == pipe(pipefd1));
        assert(0 == pipe(pipefd2));
        pipes[i][j][se][0] = pipefd1[0];
        pipes[i][j][se][1] = pipefd2[1];
        pipes[i+1][j+1][nw][0] = pipefd2[0];
        pipes[i+1][j+1][nw][1] = pipefd1[1];
          }
        else
          {
        setboundary(pipes[i][j][se]);
          }
      }
    if (j == N-1)
      {
        setboundary(pipes[i][j][s]);
      }
    else
      {
        int pipefd1[2];
        int pipefd2[2];
        assert(0 == pipe(pipefd1));
        assert(0 == pipe(pipefd2));
        pipes[i][j][s][0] = pipefd1[0];
        pipes[i][j][s][1] = pipefd2[1];
        pipes[i][j+1][w][0] = pipefd2[0];
        pipes[i][j+1][w][1] = pipefd1[1];
      }
      }
}

int main()
{
  int i, j, k;
  initpipes();

  srandom(time(NULL));
  for (i=0;i<M;i++)
    {
      for (j=0;j<N;j++)
    {
      pid_t r;
      char nr = random()%4 == 0;
      r = fork();
      assert(r != -1);
      if (r == 0)
          tick(nr, pipes[i][j]);
    }
    }

  while (1)
    for (j=0;j<N;j++)
      {
    for (i=0;i<M;i++)
      {
        char alive;
        int b;
        do {
          b = read(pipes[i][j][display][0], &alive, 1);
        } while (b != 1);
        putchar(alive ? '#' : ' ');
      }
    putchar('\n');
      }
}
