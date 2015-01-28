# Grammar for Decorator Syntax

&emsp;&emsp;*DecoratorList*<sub> [Yield]</sub>&emsp;:  
&emsp;&emsp;&emsp;*DecoratorList*<sub> [?Yield]opt</sub>&emsp; *Decorator*<sub> [?Yield]</sub>

&emsp;&emsp;*Decorator*<sub> [Yield]</sub>&emsp;:  
&emsp;&emsp;&emsp;`@`&emsp;*AssignmentExpression*<sub> [?Yield]</sub>

&emsp;&emsp;*PropertyDefinition*<sub> [Yield]</sub>&emsp;:  
&emsp;&emsp;&emsp;*IdentifierReference*<sub> [?Yield]</sub>  
&emsp;&emsp;&emsp;*CoverInitializedName*<sub> [?Yield]</sub>  
&emsp;&emsp;&emsp;*PropertyName*<sub> [?Yield]</sub>&emsp; `:`&emsp;*AssignmentExpression*<sub> [In, ?Yield]</sub>  
&emsp;&emsp;&emsp;*DecoratorList*<sub> [?Yield]opt</sub>&emsp;*MethodDefinition*<sub> [?Yield]</sub>

&emsp;&emsp;*CoverMemberExpressionSquareBracketsAndComputedPropertyName*<sub> [Yield]</sub>&emsp;:  
&emsp;&emsp;&emsp;`[`&emsp;*Expression*<sub> [In, ?Yield]</sub>&emsp;`]`

NOTE	The production *CoverMemberExpressionSquareBracketsAndComputedPropertyName* is used to cover parsing a *MemberExpression* that is part of a *Decorator* inside of an *ObjectLiteral* or *ClassBody*, to avoid lookahead when parsing a decorator against a *ComputedPropertyName*. 

&emsp;&emsp;*PropertyName*<sub> [Yield, GeneratorParameter]</sub>&emsp;:  
&emsp;&emsp;&emsp;*LiteralPropertyName*  
&emsp;&emsp;&emsp;[+GeneratorParameter] *CoverMemberExpressionSquareBracketsAndComputedPropertyName*  
&emsp;&emsp;&emsp;[~GeneratorParameter] *CoverMemberExpressionSquareBracketsAndComputedPropertyName*<sub> [?Yield]</sub>

&emsp;&emsp;*MemberExpression*<sub> [Yield]</sub>&emsp; :  
&emsp;&emsp;&emsp;[Lexical goal *InputElementRegExp*] *PrimaryExpression*<sub> [?Yield]</sub>  
&emsp;&emsp;&emsp;*MemberExpression*<sub> [?Yield]</sub>&emsp;*CoverMemberExpressionSquareBracketsAndComputedPropertyName*<sub> [?Yield]</sub>  
&emsp;&emsp;&emsp;*MemberExpression*<sub> [?Yield]</sub>&emsp;`.`&emsp;*IdentifierName*  
&emsp;&emsp;&emsp;*MemberExpression*<sub> [?Yield]</sub>&emsp;*TemplateLiteral*<sub> [?Yield]</sub>  
&emsp;&emsp;&emsp;*SuperProperty*<sub> [?Yield]</sub>  
&emsp;&emsp;&emsp;*NewSuper*&emsp;*Arguments*<sub> [?Yield]</sub>  
&emsp;&emsp;&emsp;`new`&emsp;*MemberExpression*<sub> [?Yield]</sub>&emsp;*Arguments*<sub> [?Yield]</sub>

&emsp;&emsp;*SuperProperty*<sub> [Yield]</sub>&emsp;:  
&emsp;&emsp;&emsp;`super`&emsp;*CoverMemberExpressionSquareBracketsAndComputedPropertyName*<sub> [?Yield]</sub>  
&emsp;&emsp;&emsp;`super`&emsp;`.`&emsp;*IdentifierName*

&emsp;&emsp;*ClassDeclaration*<sub> [Yield, Default]</sub>&emsp;:  
&emsp;&emsp;&emsp;*DecoratorList*<sub> [?Yield]opt</sub>&emsp;`class`&emsp;*BindingIdentifier*<sub> [?Yield]</sub>&emsp;*ClassTail*<sub> [?Yield]</sub>  
&emsp;&emsp;&emsp;[+Default] *DecoratorList*<sub> [?Yield]opt</sub>&emsp;`class`&emsp;*ClassTail*<sub> [?Yield]</sub>

&emsp;&emsp;*ClassExpression*<sub> [Yield, GeneratorParameter]</sub>&emsp;:  
&emsp;&emsp;&emsp;*DecoratorList*<sub> [?Yield]opt</sub>&emsp;`class`&emsp;*BindingIdentifier*<sub> [?Yield]opt</sub>&emsp;*ClassTail*<sub> [?Yield, ?GeneratorParameter]</sub>

&emsp;&emsp;*ClassElement*<sub> [Yield]</sub>&emsp;:  
&emsp;&emsp;&emsp;*DecoratorList*<sub> [?Yield]opt</sub>&emsp;*MethodDefinition*<sub> [?Yield]</sub>  
&emsp;&emsp;&emsp;*DecoratorList*<sub> [?Yield]opt</sub>&emsp;`static`&emsp;*MethodDefinition*<sub> [?Yield]</sub>

