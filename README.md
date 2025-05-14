# temp
/* Infix notation calculator with operator precedence and parentheses */

%{
#define YYSTYPE double
#include <stdio.h>
#include <math.h>
#include <ctype.h>

int yylex(void);
void yyerror(const char *s);
%}

/* Operator precedence: from lowest to highest */
%left '+' '-'
%left '*' '/'
%right '^'
%right UMINUS  /* Unary minus */

%token NUM

%%

input:
      /* empty */
    | input line
    ;

line:
      '\n'
    | exp '\n'   { printf("\t%.10g\n", $1); }
    ;

exp:
      NUM                 { $$ = $1; }
    | exp '+' exp         { $$ = $1 + $3; }
    | exp '-' exp         { $$ = $1 - $3; }
    | exp '*' exp         { $$ = $1 * $3; }
    | exp '/' exp         { $$ = $1 / $3; }
    | exp '^' exp         { $$ = pow($1, $3); }
    | '-' exp %prec UMINUS { $$ = -$2; }
    | '(' exp ')'         { $$ = $2; }
    ;

%%

/* Lexical analyzer: handles numbers, operators, and comments */
int yylex(void) {
    int c;

    while (1) {
        c = getchar();

        // Skip whitespace
        if (c == ' ' || c == '\t') continue;

        // Skip comments: /* ... */
        if (c == '/') {
            int next = getchar();
            if (next == '*') {
                // Inside comment
                while (1) {
                    c = getchar();
                    if (c == EOF) return 0;
                    if (c == '*') {
                        if ((c = getchar()) == '/') break;
                        else ungetc(c, stdin);
                    }
                }
                continue;
            } else {
                ungetc(next, stdin);
                return '/'; // Return division operator
            }
        }

        // Numbers
        if (isdigit(c) || c == '.') {
            ungetc(c, stdin);
            scanf("%lf", &yylval);
            return NUM;
        }

        if (c == EOF) return 0;

        return c; // Return operator or parenthesis
    }
}

/* Error handler */
void yyerror(const char *s) {
    fprintf(stderr, "Hata: %s\n", s);
}

/* Entry point */
int main(void) {
    return yyparse();
}
