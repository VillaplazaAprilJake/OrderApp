# SOLID Refactor — Order Form

## What problem each principle solved

**S — Single Responsibility.** Ang original nga Form1.cs kay daghan kaayog gibuhat sulod sa button click methods.
 Siya ang nagkuha sa data sa grid, nag compute sa discount, nag connect sa database, nag send og email, ug naghimo sa confirmation message.

### Open/Closed Principle (OCP)

Sa una, ang discount kay gipili gamit og if/else sample if (discount == "Student"). Kada naay bag-ong promotion, kinahanglan usbon ang existing code.
so kada discount naa nay have their own class nga nag implement sa IDiscountStrategy. Ang DiscountStrategyFactory mao ang nagbuot kung unsang discount ang gamiton.

### Liskov Substitution Principle (LSP)

this part is purposely breaking the assignment to see kung unsa ang mahitabo kung dili masunod ang LSP.

Ang FreeShippingDiscount gi-register uban sa ubang discount strategies. Kung pilion ang FreeShipping sa combo box unya i-click ang Calculate, then i-call gihapon sa OrderCalculator ang .Apply(subtotal).
Pero ang FreeShippingDiscount dili mo-return og discount amount. Instead, mo-throw siya og NotSupportedException. Tungod kay walay mo catch sa error, mo crash ang app.

So this is the problem of LSP: If one class is implementing the IDiscountStrategy, so then magamit siya same on the other discount strategies. 

### Interface Segregation Principle (ISP)
So its like a big interface nga nag-combine sa pricing, database, ug email.
then it divided sa tulo ka gagmay nga interfaces

* IDiscountStrategy – para sa discount
* IOrderRepository – para sa pag-save sa order
* IInvoiceSender – para sa pag-send sa email

### Dependency Inversion Principle (DIP)

Before, `Form1` itself created the `SqlConnection` and `SmtpClient`. This means it was directly connected to the SQL Server and email server.

Now, `Form1` only depends on `IOrderRepository` and `IInvoiceSender`. The actual implementations, such as `SqlOrderRepository` and `EmailInvoiceSender`, are set up in one place.

Because of this, it is easier to replace the database or email service. It is also easier to test because fake versions can be used instead of connecting to the actual database or email server.
### Part 3 – LSP Violation

`FreeShippingDiscount` was added as a discount strategy even though it is not actually a proper discount for the order total.

If it is selected and **Calculate** is clicked, `OrderCalculator` will call it using `.Apply(subtotal)`. However, instead of returning a value, it throws a `NotSupportedException`.

This is an example of breaking the LSP (Liskov Substitution Principle). The program expects every `IDiscountStrategy` to provide a discounted total, but `FreeShippingDiscount` cannot do that.

Because of this, the application crashes. This shows why LSP is important: if a class implements an interface, it should be able to follow the expected behavior of that interface.
### Part 5 – FakeOrderRepository
Useful ni para sa testing kay dili na kinahanglan mag connect sa real SQL Server. Pwede nato ma-check kung sakto ba ang Form1 sa paghimo ug pag-save sa Order.
Mas dali mas paspas ug pwede mag test bisan walay internet or database. Dili pud siya magbilin og test data sa actual database.
