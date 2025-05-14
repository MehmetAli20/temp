#-------pcalc.y------
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

#========icalc.y=========
%{
#define YYSTYPE double
#include <stdio.h>
#include <math.h>
#include <ctype.h>

int yylex(void);
void yyerror(const char *s);
%}

/* Operatör öncelikleri (düşükten yükseğe) */
%left '+' '-'
%left '*' '/'
%right '^'
%right UMINUS

%token NUM

%%

input:
      /* empty */
    | input line
    ;

line:
      '\n'
    | exp '\n'        { printf("\t%.10g\n", $1); }
    | error '\n'      { yyerror("Geçersiz ifade!"); yyerrok; }
    ;

exp:
      NUM                     { $$ = $1; }
    | exp '+' exp             { $$ = $1 + $3; }
    | exp '-' exp             { $$ = $1 - $3; }
    | exp '*' exp             { $$ = $1 * $3; }
    | exp '/' exp             {
                                if ($3 == 0) {
                                    fprintf(stderr, "Hata: Sıfıra bölme hatası! Payda: %g\n", $3);
                                    $$ = 0;
                                } else {
                                    $$ = $1 / $3;
                                }
                              }
    | exp '^' exp             { $$ = pow($1, $3); }
    | '-' exp %prec UMINUS    { $$ = -$2; }
    | '(' exp ')'             { $$ = $2; }
    ;

%%

int yylex(void) {
    int c;

    while (1) {
        c = getchar();

        // Skip whitespace
        if (c == ' ' || c == '\t')
            continue;

        // Skip comments: /* ... */
        if (c == '/') {
            int next = getchar();
            if (next == '*') {
                int prev = 0;
                while (1) {
                    c = getchar();
                    if (c == EOF) return 0;

                    if (prev == '*' && c == '/') {
                        break; // End of comment
                    }
                    prev = c;
                }
                continue; // Skip to next loop
            } else {
                ungetc(next, stdin);
                return '/'; // It was just a '/'
            }
        }

        // Process numbers
        if (isdigit(c) || c == '.') {
            ungetc(c, stdin);
            if (scanf("%lf", &yylval) != 1) {
                yyerror("Sayı okuma hatası.");
                return 0;
            }
            return NUM;
        }

        // End of input
        if (c == EOF)
            return 0;

        return c; // Return character as token
    }
}

void yyerror(const char *s) {
    fprintf(stderr, "Hata: %s\n", s);
}

int main(void) {
    return yyparse();
}

