CS 320 Principles of Programming Languages
Homework 4
Due Wednesday 3/14 before class

This homework comes with a Haskell implementation of the applied call-by-value
lambda calculus that's covered in the week 7 and 8 lecture slides.

  - Lambda.lhs is the abstract syntax definition; you won't have to modify
    this, but you should look over it to see the AST types for lambda calculus
    types and expressions.

  - LambdaLex.hs implements a lexer and LambdaParse.hs implements a parser for
    the syntax used in the lecture slides; we're done focusing on syntax in
    this class, so you don't have to look at these unless you want to. The
    important thing to know is that backslashes can be used as lambdas, but
    since the backslash is also the escape character in Haskell strings, you
    have to use two backslashes every time (e.g. "(\\x. x + x) 1").

  - LambdaSmallStep.lhs implements the small-step semantics from the week 7
    lecture slides. The "readEval" function can be used to evaluate strings
    expressions written as strings.

  - LambdaBigStep.lhs implements the big-step semantics from the week 8 lecture
    slides. Like in the small step file, there's a "readEval" function that can
    be used to evaluate expressions written as strings; see the bottom of the
    file for example usage.

  - LambdaTypecheck.lhs implements the typechecking rules from the week 8
    lecture slides. There's a "readTypecheck" function that can be used to
    typecheck expressions written as strings; see the bottom of the file for
    example usage.

1. (8 points)

Normalize the following untyped applied lambda calculus expressions as much as
possible using the call-by-value rules, showing all steps. You can either do
this by hand or extend LambdaSmallStep.hs with a function to do it for you; if
you do it with a function, make sure to include the function in your
submission. (Hint: the final result of each normalization can be checked with
the provided "readEval" function in LambdaSmallStep.hs.)

a) (λx. x * x) 1
	= (x * x) [1/x]
	=> (1 1)
	=> (1)

b) (λx. x + 4) ((λy. y + 5) 3)
	=> (λx. x + 4) ((y + 5) [3/y])
	=> (λx. x + 4) (3 + 5)
	=> (x + 4) [(8)/x]
	=> (8) + 4
	=> 12

c) (λf g x. g (f x)) (λa. a * a) (λb. b + 1 + 2) 3
	   (((λf. (λg. (λx. g (f x)))) (λa. a * a)) (λb. b + 1 + 2)) 3
	=> ((λg. (λx. g (f x)) [(λa. a * a)/f]) (λb. b + 1 + 2)) 3
	=> ((λg. (λx. g ((λa. a * a) x))) (λb. b + 1 + 2)) 3
	=> ((λx. g ((λa. a * a) x)) [(λb. b + 1 + 2)/g]) 3
	=> (λx. (λb. b + 1 + 2) ((λa. a * a) x))  3
	=> ((λb. b + 1 + 2) ((λa. a * a) x)) [3/x]
	=> (λb. b + 1 + 2) ((λa. a * a) 3)
	=> (λb. b + 1 + 2) ((a * a) [3/a])
	=> (λb. b + 1 + 2) (3 * 3)
	=> (λb. b + 1 + 2) 9
	=> (b + 1 + 2) [9/b]
	=> (9 + 1 + 2)
	=> 12

d) (λf x. f (f (f x))) (λb. if b then false else true) true
	   (λf (λx. f (f (f x))) (λb. if b then false else true)) true
	=> (λx. f (f (f x)) [(λb. if b then false else true)/f]) true
	=> (λx. (λb. if b then false else true) ((λb. if b then false else true) ((λb. if b then false else true) x))) true
	=> ((λb. if b then false else true) ((λb. if b then false else true) ((λb. if b then false else true) x)) [true/x])
	=> ((λb. if b then false else true) ((λb. if b then false else true) ((λb. if b then false else true))) true)
	=> ((λb. if b then false else true) ((λb. if b then false else true) ((λb. if b then false else true))) [true/b]
	=> (if true then false else true) (if true then false else true) (if true then false else true)
	=> if true then false else true

	




2. (8 points)

For each of the following simply-typed applied lambda calculus expressions,
give a typing derivation or show that the expression is ill-typed by showing
that there can't exist a typing derivation for it. (Hint: the final result of
typechecking each expression can be checked with the provided "readTypecheck"
function in LambdaTypecheck.hs.)

a) if (if true then false else true) then false else true

		true:bool	false:bool	true: bool	false:bool	true:bool
T-If--------------------------------------------------------------
		if true then false else true:bool	false:bool	true:bool
T-If--------------------------------------------------------------
		∅ ⊢ if(if true then false else true) then false else true

b) λ(x: num). x + x * 3

			x:num	x:num	x:num	3:num
T-Times--------------------------------------
			x:num 	x:num	x:num	* 3:num
T-Plus---------------------------------------
			x:num ⊢	x + x * 3:num
T-Lam----------------------------------------
			∅ ⊢	λ(x: num). x + x * 3

c) λ(a: bool). if a then 1 else a

		a:bool, a:bool, 1:bool,	a:bool
T-If-----------------------------------------
		a:bool, if a then 1 else, a:bool
T-Lam----------------------------------------
		∅ ⊢	λ(a: bool). if a then 1 else a

d) (λ(f: num -> bool) (a: num). if f a then a + 1 else a + 2) (λ(x: num). false)

		f:num, a:num. ⊢ f:bool a:bool ⊢ a:bool, ⊢ 1:bool,  a:bool, ⊢ 2:bool, false:bool
T-Plus--------------------------------------------------------------------------------------
		f:num, a:num. ⊢ f:bool a:bool ⊢ a + 1:bool  ⊢ a + 2:bool, false:bool
T-If--------------------------------------------------------------------------------------
		f:num, a:num. ⊢ if f a then a + 1 else a + 2:bool,	false:bool
T-App--------------------------------------------------------------------------------------
		f:(num -> bool):num, a:num. ⊢ if f a then a + 1 else a + 2:num,	false:bool
T-Lam--------------------------------------------------------------------------------------
		f:(num -> bool):num ⊢ (λ(a:num). if f a then a + 1 else a + 2):num,	false:bool
T-Lam--------------------------------------------------------------------------------------
		∅ ⊢ (λ(f: num -> bool) (a: num). if f a then a + 1 else a + 2) (λ(x: num). false


3. (20 points)

At each of the five commented lines in the following procedural program:
  - Write out the contents of the type environment during typechecking.

  - Write out the contents of the evaluation environment (the stack) during
    evaluation for each time the line is hit during execution, assuming
    execution starts with main().

num distance(num x, num y) {
  bool b = x < y;
  num d = 0;
  // check: 0:num		stack: b:bool, x:num, y:num, d:num, m:num

  if b {
    num z = y - x;
    d = z;
    // check: z:num		stack: b:bool, x:num, y:num, d:num, m:num
  } else {
    num w = x - y;
    d = w;
    // check: w:num		stack: b:bool, x:num, y:num, d:num, m:num, w:num
  }

  // 	check: none		stack: b:bool, x:num, y:num, d:num, m:num
  return m;
}

unit main() {
  num x = distance(1, 2);
  num y = distance(4, 3);
  // check distance(4, 3) is well-typed    Stack: x:num,  y:num, distance: num -> num -> num
}


4. (4 points)

Add a case for the numeric less-than operator (the Less constructor) to the
"step" function in LambdaSmallStep.lhs, the "eval" function in
LambdaBigStep.lhs, or the "typecheck" function in LambdaTypecheck.lhs. (The
parsing code for the operator is already implemented.)

You're only required to add the case in one file of your choice: there are 12
points available in this question (4 for each file) but the question is graded
out of 4 points, so if you add it correctly in more than one file it'll count
as extra credit.

Here are the inference rules for the operator in each semantics:

//did typechecking.
//started small step on bottom of the file.


Small step:

         n1 < n2                   n1 >= n2
    ------------------        -------------------
     (n1 < n2) ⇒ true          (n1 < n2) ⇒ false
 
 
         e1 ⇒ e1'                   e2 ⇒ e2'
 ------------------------   ------------------------
  (e1 < e2) ⇒ (e1' < e2)     (v1 < e2) ⇒ (v1 < e2')



Big step:

  e1 ⇓ n1   e2 ⇓ n2   n1 < n2     e1 ⇓ n1   e2 ⇓ n2   n1 >= n2
 -----------------------------   ------------------------------
     <ρ, (e1 < e2)> ⇓ true           <ρ, (e1 < e2)> ⇓ false



Typechecking:

  Γ ⊢ e1 : num   Γ ⊢ e2 : num
 -----------------------------
      Γ ⊢ (e1 < e2) : bool
