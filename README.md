%{
#define YYSTYPE double
#include <stdio.h>
#include <math.h>
#include <ctype.h>

int yylex(void);
void yyerror(const char *s);
%}

%token NUM

%%

input:
    /* boş */
  | input line
  ;

line:
    '\n'
  | exp '\n' { printf("\t%.10g\n", $1); }
  ;

exp:
    NUM             { $$ = $1; }
  | exp exp '+'     { $$ = $1 + $2; }
  | exp exp '-'     { $$ = $1 - $2; }
  | exp exp '*'     { $$ = $1 * $2; }
  | exp exp '/'     {
                      if ($2 == 0) {
                        fprintf(stderr, "Hata: Sıfıra bölme! Payda: %g\n", $2);
                        $$ = 0;
                      } else {
                        $$ = $1 / $2;
                      }
                    }
  | exp exp '^'     { $$ = pow($1, $2); }
  | exp 'n'         { $$ = -$1; }
  ;

%%

int yylex(void) {
    int c;

    while (1) {
        c = getchar();

        // Boşluk ve tab atla
        if (c == ' ' || c == '\t') continue;

        // Yorum satırı atla: /* ... */
        if (c == '/') {
            int next = getchar();
            if (next == '*') {
                int prev = 0;
                while (1) {
                    c = getchar();
                    if (c == EOF) return 0;
                    if (prev == '*' && c == '/') break;
                    prev = c;
                }
                continue;
            } else {
                ungetc(next, stdin);
                return '/';
            }
        }

        // Sayı
        if (isdigit(c) || c == '.') {
            ungetc(c, stdin);
            scanf("%lf", &yylval);
            return NUM;
        }

        if (c == EOF) return 0;

        return c;
    }
}

void yyerror(const char *s) {
    fprintf(stderr, "Hata: %s\n", s);
}

int main(void) {
    return yyparse();
}
