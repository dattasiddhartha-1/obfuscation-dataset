#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/time.h>
#include <signal.h>

#define NEXT_LINE 21

#define CURVE_LEFT 0
#define CURVE_NONE 1
#define CURVE_RIGHT 2

char x_offset[] = {
  40, 40, 40, 40, 40, 40, 40, 40, 40, 40,
  40, 40, 40, 40, 40, 40, 40, 40, 40, 40,
  40, 40, 0 };

char width[] = {
  8, 8, 8, 8, 8, 8, 8, 8, 8, 8,
  8, 8, 8, 8, 8, 8, 8, 8, 8, 8,
  8, 8, 0 };

const char border[] = "\\|/";

void change_bell_frequency () {}
void clear_screen () {}
void clear_all_LEDs () {}
void set_Num_Lock_LED () {}
void set_Scroll_lock_LED () {}
void set_Caps_Lock_LED () {}



char *tree[] = {
  "",
  " %%%",
  "%\\|/%",
  "  |",
  "  |",
};


char **tree_line = tree;
char tree_position;

char curve = CURVE_NONE;
char *a, y, z;

char move_x = 0;
char update_interval = -1;

char pos = 40;
char old_pos = 40;

char play_sound = 0;
char gear;

unsigned int score = 0;

void move (char x, char y) {
  printf ("\e[%u;%uH", x, y);
}

void insert () {
  printf ("\e[L");
}

void update (int i) {
  int n;

  pos += move_x,

  /* draw street */
  printf ("\e[1;%uH" "\e[L" "%c"
          "\e[1;%uH" "%c",
          *x_offset - *width, border[curve],
          *x_offset + *width, border[curve]);
  /* draw tree */
  printf ("\e[1;%uH" "%s",
          tree_position, *tree_line);

  /* redraw car */
  printf ("\e[%u;%uH" "@"
          "\e[%u;%uH" " " "\n",
          NEXT_LINE + 1, pos,
          NEXT_LINE +2, old_pos);

  /* did we leave the road ? */
  if (abs (pos - x_offset[NEXT_LINE]) >= width[NEXT_LINE])
    exit (0);

  /* update simulation speed */
  if (update_interval != gear) {
    struct itimerval t = { 0, 0, 0, 0 }  ;
      update_interval += ((update_interval < gear) << 1) - 1;
      t.it_interval.tv_usec = t.it_value.tv_usec = 72000 / ((update_interval >> 3) + 1);
      setitimer (0, &t, 0);
      if (play_sound)
        change_bell_frequency (update_interval + 24);
  }

  /* play sound */
  if (play_sound)
    putchar ('\a');

  /* update score */
  score += (9 - width[NEXT_LINE]) * ((update_interval >> 3) + 1);
  old_pos = pos;

  /* shift x_offset */
  a = x_offset;
  z = *a;
  while (*++a) {
    y = *a;
    *a = z;
    z = y;
  };

  /* shift width */
  a = width;
  z = *a;
  while (*++a) {
    y = *a;
    *a = z;
    z = y;
  };

  /* generate new road */
  n = rand ();

  if (!(n & 255) && *width > 1)
    --*width;

  /* set tree line pointer */
  if (!(**tree_line && tree_line-- || n & 7936)) {
    /* find the right spot to grow */
    while (abs ((tree_position = rand () % 76) - *x_offset + 2) - *width < 6)
      ;
    ++tree_position;
    tree_line = &tree[4];
  }

  /* new offset */
  n = rand () & 31;
  if (n < 3)
    curve = n;

  if (curve == CURVE_LEFT) {
    --*x_offset;
    if (*x_offset <= *width) {
      ++*x_offset;
      curve = CURVE_NONE;
    }
  }
  else if (curve == CURVE_RIGHT) {
    ++*x_offset;
    if (*x_offset + *width > 79) {
      --*x_offset;
      curve = CURVE_NONE;
    }
  }

  signal (SIGALRM, update);
}


void end () {
  signal (SIGALRM, SIG_IGN);
  clear_all_LEDs ();
  clear_screen ();
  printf ("Score: %u\n", score);
  system ("stty echo -cbreak");
}


int main (int argc, char **argv) {
  atexit (end);

  if (argc < 2 || *argv[1] != 'q') {
    argc = *(int*) getenv ("TERM");
    if (argc == (int) 0x6C696E75 || argc == (int) 0x756E696C)
      play_sound = 1;
  }

  srand (getpid ());
  system ("stty -echo cbreak");
  gear = 0 << 3;

  clear_all_LEDs ();
  update (14);
  for (;;)
    switch (getchar ())
      {
        case 'q':
          return 0;
        case '[':
        case 'b':
        case ',':
          move_x = -1;
          continue;
        case ' ':
        case 'n':
        case '.':
          move_x = 0;
          continue;
        case ']':
        case 'm':
        case '/':
          move_x = 1;
          continue;
        case '1':
          gear = 0 << 3;
          set_Num_Lock_LED ();
          continue;
        case '2':
          gear = 1 << 3;
          set_Caps_Lock_LED ();
          continue;
        case '3':
          gear = 2 << 3;
          set_Scroll_lock_LED ();
          continue;
        case '4':
          gear = 3 << 3;
          clear_all_LEDs ();
          continue;
      }
}