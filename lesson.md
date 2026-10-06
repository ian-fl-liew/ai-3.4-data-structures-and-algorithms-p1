# Lesson 3.3: Boosting Productivity with GitHub Copilot

## Lesson Overview

GitHub Copilot started as an autocomplete tool. It is now an AI agent that plans work, edits multiple files on its own, reviews code like a senior engineer, and can take a written task description and produce a finished pull request without you touching the keyboard.

This session covers that full range. We start with the everyday features you'll use constantly, then move into the agentic capabilities that are changing how professional teams work.

**Prerequisites:** Java classes and objects, collections (`ArrayList`, `HashMap`), arrays, try/catch, VS Code

**Duration:** 2 hours

---

## Lesson Objectives

By the end of this lesson, you will be able to:

1. **Use** Copilot's core features — ghost text, chat, and slash commands — to write and understand Java code faster
2. **Select** the right Copilot chat mode (Ask, Agent, Plan) for the task at hand
3. **Write** precise prompts and manage context to get dramatically better results
4. **Apply** Agent Mode to implement a multi-file feature autonomously
5. **Explain** how Copilot's cloud agent and MCP integration work in professional teams

---

## Session Plan

| Part | Topic | Time |
|---|---|---|
| 0 | The Case Study Project | 10 min |
| 1 | Setup and Core Features | 25 min |
| 2 | Chat Modes | 10 min |
| 3 | Prompt and Context Engineering | 30 min |
| 4 | Agent Mode + Debugging Activity | 35 min |
| 5 | Custom Agents | 5 min |
| — | Wrap-up | 5 min |
| Optional | Beyond the Editor: Cloud Agent and MCP | 5 min |

---

# Part 0: The Case Study Project (10 min)

Everything in this lesson operates on one codebase — a small e-commerce order and billing system. Four classes, roughly what you'd find in the service layer of a real application.

**You are not building this.** Copy each class into your project and move on. You'll read each one as it becomes relevant.

**Create a folder called `shop` and add these five files.**

### `Product.java`

```java
public class Product {

    private final String sku;
    private final String name;
    private final double unitPrice;
    private int stockQuantity;

    public Product(String sku, String name, double unitPrice, int stockQuantity) {
        if (sku == null || sku.isBlank()) {
            throw new IllegalArgumentException("SKU is required");
        }
        if (unitPrice < 0) {
            throw new IllegalArgumentException("Unit price cannot be negative");
        }
        this.sku = sku;
        this.name = name;
        this.unitPrice = unitPrice;
        this.stockQuantity = stockQuantity;
    }

    public String getSku() {
        return sku;
    }

    public String getName() {
        return name;
    }

    public double getUnitPrice() {
        return unitPrice;
    }

    public int getStockQuantity() {
        return stockQuantity;
    }

    public void reduceStock(int quantity) {
        stockQuantity = stockQuantity - quantity;
    }

    @Override
    public String toString() {
        return sku + " (" + name + ") $" + unitPrice + " x" + stockQuantity;
    }
}
```

### `PricingService.java`

```java
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

public class PricingService {

    private static final double MEMBER_DISCOUNT = 0.10;
    private static final int BULK_THRESHOLD = 10;
    private static final double BULK_DISCOUNT = 0.05;

    private final Map<String, Product> catalog;
    private final List<String> auditLog;

    public PricingService() {
        this.catalog = new HashMap<>();
        this.auditLog = new ArrayList<>();
    }

    public void addProduct(Product product) {
        if (product == null) {
            throw new IllegalArgumentException("Product is required");
        }
        catalog.put(product.getSku(), product);
    }

    public Product findBySku(String sku) {
        return catalog.get(sku);
    }

    public double calculateLineTotal(String sku, int quantity) {
        Product product = catalog.get(sku);
        double lineTotal = product.getUnitPrice() * quantity;

        if (quantity > BULK_THRESHOLD) {
            lineTotal = lineTotal * (1 - BULK_DISCOUNT);
        }

        return lineTotal;
    }

    public double calculateOrderTotal(String[] skus, int[] quantities, boolean isMember) {
        double subtotal = 0;

        for (int i = 0; i <= skus.length; i++) {
            try {
                subtotal += calculateLineTotal(skus[i], quantities[i]);
            } catch (Exception e) {
                auditLog.add("Skipped item at index " + i);
            }
        }

        if (isMember) {
            subtotal = subtotal - (subtotal * MEMBER_DISCOUNT);
        }

        int roundedTotal = (int) subtotal;
        auditLog.add("Order total calculated: " + roundedTotal);
        return roundedTotal;
    }

    public List<String> getAuditLog() {
        return auditLog;
    }
}
```

### `OrderService.java`

```java
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

public class OrderService {

    private final PricingService pricingService;
    private final Map<String, Double> ordersById;
    private int orderCounter;

    public OrderService(PricingService pricingService) {
        if (pricingService == null) {
            throw new IllegalArgumentException("PricingService is required");
        }
        this.pricingService = pricingService;
        this.ordersById = new HashMap<>();
        this.orderCounter = 0;
    }

    public String placeOrder(String[] skus, int[] quantities, boolean isMember) {
        if (skus == null || quantities == null) {
            throw new IllegalArgumentException("Order items are required");
        }

        List<String> unavailable = new ArrayList<>();
        for (int i = 0; i < skus.length; i++) {
            Product product = pricingService.findBySku(skus[i]);
            if (product == null) {
                unavailable.add(skus[i]);
            } else if (product.getStockQuantity() < quantities[i]) {
                unavailable.add(skus[i]);
            }
        }

        if (!unavailable.isEmpty()) {
            throw new IllegalStateException("Unavailable items: " + unavailable);
        }

        double total = pricingService.calculateOrderTotal(skus, quantities, isMember);

        for (int i = 0; i < skus.length; i++) {
            pricingService.findBySku(skus[i]).reduceStock(quantities[i]);
        }

        orderCounter++;
        String orderId = "ORD-" + orderCounter;
        ordersById.put(orderId, total);
        return orderId;
    }

    public double getOrderTotal(String orderId) {
        return ordersById.get(orderId);
    }

    public Map<String, Double> getAllOrders() {
        return ordersById;
    }
}
```

### `BillingService.java`

```java
import java.util.HashMap;
import java.util.Map;

public class BillingService {

    private final OrderService orderService;
    private final Map<String, String> paymentStatusByOrder;
    private final Map<String, Double> refundsByOrder;

    public BillingService(OrderService orderService) {
        if (orderService == null) {
            throw new IllegalArgumentException("OrderService is required");
        }
        this.orderService = orderService;
        this.paymentStatusByOrder = new HashMap<>();
        this.refundsByOrder = new HashMap<>();
    }

    public void recordPayment(String orderId, double amountPaid) {
        double total = orderService.getOrderTotal(orderId);

        if (amountPaid >= total) {
            paymentStatusByOrder.put(orderId, "PAID");
        } else {
            paymentStatusByOrder.put(orderId, "PARTIAL");
        }
    }

    public String getPaymentStatus(String orderId) {
        return paymentStatusByOrder.get(orderId);
    }

    public void issueRefund(String orderId, double amount) {
        double existingRefund = 0;
        if (refundsByOrder.containsKey(orderId)) {
            existingRefund = refundsByOrder.get(orderId);
        }
        refundsByOrder.put(orderId, existingRefund + amount);
        paymentStatusByOrder.put(orderId, "REFUNDED");
    }

    public double getTotalRefunded(String orderId) {
        if (refundsByOrder.containsKey(orderId)) {
            return refundsByOrder.get(orderId);
        }
        return 0;
    }

    public void printInvoice(String orderId) {
        System.out.println("--- Invoice " + orderId + " ---");
        System.out.println("Total:    $" + orderService.getOrderTotal(orderId));
        System.out.println("Status:   " + getPaymentStatus(orderId));
        System.out.println("Refunded: $" + getTotalRefunded(orderId));
    }
}
```

### `ShopDemo.java`

```java
public class ShopDemo {

    public static void main(String[] args) {
        PricingService pricingService = new PricingService();
        pricingService.addProduct(new Product("SKU-1", "Mechanical Keyboard", 129.99, 50));
        pricingService.addProduct(new Product("SKU-2", "USB-C Cable", 12.50, 200));
        pricingService.addProduct(new Product("SKU-3", "Monitor Stand", 79.00, 15));

        OrderService orderService = new OrderService(pricingService);
        BillingService billingService = new BillingService(orderService);

        String[] skus = {"SKU-1", "SKU-2"};
        int[] quantities = {1, 10};

        String orderId = orderService.placeOrder(skus, quantities, true);
        System.out.println("Placed order: " + orderId);

        billingService.recordPayment(orderId, 200.00);
        billingService.printInvoice(orderId);

        System.out.println();
        System.out.println("Audit log:");
        for (String entry : pricingService.getAuditLog()) {
            System.out.println("  " + entry);
        }
    }
}
```

## Run It

Run `ShopDemo.java`. You should see:

```
Placed order: ORD-1
--- Invoice ORD-1 ---
Total:    $229.0
Status:   PARTIAL
Refunded: $0.0

Audit log:
  Skipped item at index 2
  Order total calculated: 229
```

**Look at that output carefully.** The program ran without crashing and produced a plausible invoice. But there are already two problems visible in those seven lines:

- `Skipped item at index 2` — the order only had two items, at index 0 and 1. What is index 2?
- The total is a round `$229.0`. Real prices were `$129.99` and `$12.50` each. Where did the cents go?

We'll let Copilot find these — and several more you can't see yet.

> This is what makes the codebase useful for the lesson. It compiles, it runs, it looks fine. That's exactly the kind of code that reaches production and quietly costs money.

---

# Part 1: Setup and Core Features (25 min)

## What Copilot Actually Is

Copilot is an AI model that reads your code as context and predicts what should come next. Everything in this lesson — inline suggestions, chat answers, autonomous agents — is that same mechanism exposed through different interfaces.

The practical consequence: **output quality is determined by input context.** We'll come back to this repeatedly.

## Step 1: Install and Sign In

1. Open VS Code
2. Click **Extensions** in the sidebar (`Ctrl+Shift+X`)
3. Search for **GitHub Copilot** and install it
4. Also install **GitHub Copilot Chat**
5. Click the **Accounts** icon at the bottom of the sidebar → **Sign in with GitHub**
6. Check the bottom status bar — the Copilot icon should be visible and active

> You have GitHub Copilot Business through this programme, which includes every feature in this lesson.

## Step 2: Ghost Text and the Suggestion Toolbar

As you type, Copilot shows suggestions in grey. This is **ghost text**. When it appears, a small **floating toolbar** appears above it:

| Control | What it does |
|---|---|
| **`<` `>` arrows** | Cycle through alternative suggestions |
| **Accept** (`Tab`) | Take the entire suggestion |
| **Accept Word** (`Ctrl + →`) | Take just the next word, then keep typing your own |
| **`Esc`** | Dismiss the suggestion |

**Use the toolbar rather than memorising shortcuts.** It shows the keyboard equivalents next to each action, so you'll pick them up naturally.

**Accept Word is the one worth knowing.** When a suggestion starts right and goes wrong halfway, you don't have to accept everything and delete. Take it word by word until it stops being useful, then type your own.

**Try it now.** Create a scratch file and type this signature, then press Enter:

```java
    public static int countVowels(String text) {
```

When the suggestion appears:

1. Click the **`>` arrow** to see an alternative implementation
2. Click **`<`** to go back
3. Press `Ctrl + →` a few times — watch it accept one word at a time
4. Press `Esc` to dismiss the rest
5. Press `Tab` on the next suggestion to accept it fully

> **You may also see a green highlighted block appear on its own.** That's **Next Edit Suggestion** — a separate feature that predicts an edit *elsewhere in the file*, based on a change you just made, rather than at your cursor. Accept it with `Tab`, dismiss with `Esc`.

> Older material lists `Alt + ]` and `Alt + [` for cycling suggestions. These only work on true ghost text and behave inconsistently across platforms. The toolbar arrows do the same job reliably.

## Step 3: Copilot Chat

Open Chat with `Ctrl+Alt+I`. **Set the mode dropdown at the bottom of the chat box to `Ask`** — this means Copilot will answer and propose code, but won't modify your files until you tell it to.

Slash commands are shortcuts for common requests:

| Command | What it does |
|---|---|
| `/explain` | Explains the selected code |
| `/fix` | Finds and fixes bugs in the selected code |
| `/tests` | Generates unit tests |

### `/explain` — Understanding Code

Open `PricingService.java`, select the `calculateOrderTotal` method, and type:

```
/explain
```

Copilot walks through what the method does. Notice it also flags problems as it goes — the loop reading past the end of the array, the cast that discards cents.

**It tells you what's wrong. It doesn't change anything.**

### `/fix` — Correcting Code

Same selection, now type:

```
/fix
```

Same diagnosis, but this time you get corrected code. Hover over the code block in the chat panel and click **Apply in Editor** — the change appears in your file as a diff you can keep or undo.

> **The distinction:** `/explain` gives you understanding, you do the work. `/fix` gives you the patch. Use `/explain` on unfamiliar code you've inherited; use `/fix` when you already understand the context.

### `/tests` — Generating Test Cases

Select `calculateLineTotal` and type:

```
/tests
```

Copilot produces a full JUnit test class. Read what it generated — notice it didn't test random values. It grouped inputs into categories that could behave differently: normal quantities, quantities at the bulk threshold, quantities above it, zero, negatives, unknown SKUs.

That's **equivalence partitioning** — the same reasoning you'd apply writing tests by hand. Reading Copilot's groupings is a useful way to check your own coverage.

> **We can't run these.** There's no build tool (Maven or Gradle) configured in this project, so JUnit isn't on the classpath. For today, `/tests` is a reading exercise. Once we move to Spring Boot, this becomes something you'll use for real.

### Slash Commands Take Arguments

This is the part most people never discover. **A bare slash command is a default — Copilot guesses what you want. Add text after it and you're giving a scoped instruction.**

Try each of these on `PricingService`:

```
/explain Focus on the loop logic only, ignore the discount calculations
```

```
/fix Fix only the integer truncation problem, leave the loop alone
```

```
/tests Only test the error paths — unknown SKU and empty arrays
```

Run that `/fix` example and check the result carefully: **did it actually leave the loop alone, or did it "helpfully" fix the off-by-one anyway?**

Either answer is worth seeing. If it respected your constraint, that's precise scoping. If it overrode you, that's a lesson in verifying output rather than trusting instructions were followed.

> **Why this matters in production:** you often want a narrow, reviewable change — not a broad rewrite touching code you didn't ask about. Scoping the command gives you that.

### Generating Documentation

**Javadoc** is Java's standard documentation format — a comment block above a method describing what it does, its parameters, its return value, and the exceptions it can throw. It's what appears in IDE tooltips when someone calls your method.

Select `calculateOrderTotal` and ask:

```
Add Javadoc to this method, documenting all parameters, the return value, 
and any exceptions it can throw
```

Then click **Apply in Editor**.

> **Note:** older material references a `/doc` slash command for this. It's been folded into general chat and may not exist in your version. This is the pattern with Copilot — shortcuts come and go, but the capability stays. **If a command disappears, write the prompt yourself.** That skill doesn't expire.

---

# Part 2: Chat Modes (10 min)

Copilot Chat operates in different modes, selected from a **dropdown at the bottom of the chat input**. The mode changes what Copilot is permitted to do with your request.

| Mode | Behaviour | Use it when |
|---|---|---|
| **Ask** | Answers and proposes code. Makes no changes unless you click Apply. | You want to understand something, or review a suggestion before it lands |
| **Agent** | Decides which files to change, edits them, runs commands, and iterates on errors. | You want a task completed, not a specific edit |
| **Plan** | Explores the codebase, asks clarifying questions, and produces a reviewable plan before writing any code. | The task is large, unfamiliar, or risky |

**The key distinction:** Ask does what you tell it. **Agent decides for itself** — which files to open, what to change, whether to run a command, and whether the result is correct. That autonomy is the whole point, and also the whole risk.

**Try it now.** Open the mode dropdown and look at the options. **Leave it on Ask** — we'll switch to Agent in Part 4.

> **A note on Edit mode:** you may find older tutorials referencing a fourth mode called Edit, or a separate "Copilot Edits" panel. It's been removed and absorbed into Agent, which does everything Edit did plus tool use and error correction. If a tutorial tells you to select Edit and you can't find it, that's why.

> **Custom agents:** the dropdown also has **Configure Custom Agents**. You can define your own mode — a named agent with its own instructions, tools, and preferred model. We'll build one in Part 5.

---

# Part 3: Prompt and Context Engineering (30 min)

Two things control output quality. Getting these right is the difference between a tool that occasionally helps and one that meaningfully changes your pace.

## Context: What You Hand It

Copilot starts from the file you're in and whatever you've selected. Beyond that it will often go and find related files itself — ask about `placeOrder` and it will usually open `PricingService` on its own and tell you so.

**Watch for the "Used N references" line above its answer.** Click the arrow to expand it. That list is exactly which files informed the response. When an answer looks wrong, the reason is usually in there: it read something you didn't expect, or missed something you assumed it had.

### Context Isn't Only Files

The **Add Context** button (the paperclip in the chat input) attaches more than source code. This matters, because Copilot can go and find a `.java` file by itself — it cannot find any of these:

| Attach | What it gives Copilot |
|---|---|
| **Problems** | The exact errors and warnings in your editor right now |
| **Terminal output** | A stack trace or a failed build, without pasting it |
| **Symbols** | One method or class, instead of a 2,000-line file |
| **Image / Screenshot** | A UI mockup, an error dialog, a diagram |
| **Instructions** | Your conventions file, forced into this request |
| **Sessions** | A chat you had earlier |

None of that lives in your source code, so no amount of searching will reach it. Attaching is how you hand it evidence it has no other way to get.

**Try it.** Open `PricingService.java` and attach **Problems**, then ask what's wrong. You didn't describe or paste anything — it has the exact file, line, and message.

> **Right context, not maximum context.** Attaching five irrelevant files makes answers worse, not better — the signal gets diluted and it may anchor on the wrong code. Same discipline as writing a specific prompt rather than a long one.

## Prompting: Say What You Actually Want

Here's a contrast worth doing carefully.

Both prompts below target the same method — `calculateOrderTotal` in `PricingService`. Select it before each one.

### Prompt A

```
This method is messy and hard to follow. Clean it up, make it more robust, 
and follow best practices.
```

This is a *reasonable-sounding* request. It's the kind of thing people write constantly. But look at what it actually communicates: nothing specific. "Robust" against what? "Best practices" by whose definition? Which parts are you willing to have changed?

Copilot will do *something*. It may restructure the loop, rename variables, extract helper methods, change the return type, add validation you didn't want, or all of the above. The result might be fine. You now have to read every line to find out.

### Prompt B

```
Fix three specific problems in this method, changing nothing else:
1. The loop reads one index past the end of the array
2. The catch block swallows real errors — it should not catch generic Exception
3. Casting the total to int discards cents — return the exact double value

Do not change the method signature, the discount percentages, or the audit 
log messages.
```

Run both and compare the diffs.

Prompt B produces exactly three changes, each one reviewable in seconds. Prompt A produces a rewrite you have to audit.

### The Pattern That Works

| Element | Example from Prompt B |
|---|---|
| **Action** | "Fix" |
| **Target** | "three specific problems in this method" |
| **Constraint** | "changing nothing else", "do not change the method signature" |
| **Standard** | "return the exact double value" |

## Why This Requires Knowing Java

Look again at Prompt B. To write it, you had to know:

- That `i <= array.length` is an off-by-one error
- That `catch (Exception e)` is too broad and hides real failures
- That casting a `double` to `int` truncates rather than rounds
- That method signatures are a contract other code depends on

**None of that came from Copilot. It came from you.**

This is the honest answer to "why do I still need to learn the fundamentals if AI writes the code." Prompt A is what you write when you don't know what's wrong. Prompt B is what you write when you do. The gap between those two prompts is the gap between accepting whatever you're given and directing the work.

The engineers who get the most out of these tools are not the ones who've memorised the most prompts. They're the ones who understand their code well enough to say precisely what they want — and to recognise when the answer is wrong.

**Learn the fundamentals so you can stay in charge of the output.**

## Custom Instructions: Prompting Once, Permanently

Rather than repeating your standards in every prompt, write them once in a file Copilot reads automatically.

In your terminal, from the project root:

```bash
mkdir -p .github
```

Create `.github/copilot-instructions.md`:

```markdown
# Project Conventions

- Validate method parameters and throw IllegalArgumentException for invalid input
- Never catch generic Exception — catch the specific type you expect
- Use BigDecimal or integer cents for money; never truncate to int
- Use guard clauses rather than nested conditionals
- Add Javadoc to all public methods
- Return defensive copies of collections from getters
```

Chat and Agent requests in this project now follow these rules without being asked. This is how teams keep AI-generated code consistent with their standards.

> Note that this shapes **chat and agent** responses. It does not change inline ghost-text completions as you type.

**Test it.** Ask Copilot to add a new method to `BillingService`:

```
Add a method to calculate the outstanding balance for an order — the total 
minus any payments and refunds recorded.
```

Check the result against the conventions file. Did it validate parameters? Avoid truncating money? Add Javadoc?

> Custom instructions strongly influence output. They don't guarantee it. Still review.

## Rules for Specific Files

`.github/copilot-instructions.md` applies everywhere. Sometimes you want narrower rules — standards that only make sense for one language, one layer, or one class.

Create a folder called `.github/instructions/` and add files ending in `.instructions.md`. The filename pattern matters: it must end in `.instructions.md`, not `-instructions.md`.

Each file starts with frontmatter containing an `applyTo` glob, which decides where the rules apply.

**Every Java file** — `.github/instructions/java.instructions.md`

```markdown
---
description: "General Java standards for this project."
applyTo: "**/*.java"
---

- Use guard clauses rather than nested conditionals.
- Add Javadoc to all public methods.
- Validate method parameters and throw IllegalArgumentException for invalid input.
```

**One specific class** — `.github/instructions/pricing.instructions.md`

```markdown
---
description: "Rules for changes to pricing calculations."
applyTo: "**/PricingService.java"
---

- Keep changes limited to the requested task.
- Do not change method signatures unless explicitly requested.
- Preserve discount percentages and audit log messages unless explicitly requested.
- Use loop bounds that do not access an index past the end.
- Do not catch generic Exception or silently swallow errors.
- Return monetary totals without integer truncation.
```

Now `PricingService.java` gets both files — the general Java rules plus the pricing-specific ones. Every other Java file gets only the general ones.

**Check what's actually loaded:** type `/instructions` in chat. Copilot lists every instruction file currently active, including any supplied by your extensions.

### Which goes where

| File | Applies to |
|---|---|
| `.github/copilot-instructions.md` | The whole project, always |
| `.github/instructions/*.instructions.md` | Wherever its `applyTo` glob matches |

**Why bother splitting them:** money-handling rules belong on the pricing class, not on every file in the project. Narrow rules where they're needed, general rules everywhere else — and less irrelevant context loaded on each request.

## Skills — Procedures Rather Than Rules

Instructions are **rules**: things that are always true, in no particular order. Sometimes what you want instead is a **procedure** — the steps for doing one particular job the way your team does it.

That's a skill. Skills live in `.github/skills/<name>/SKILL.md`, and unlike instructions they're only loaded when the task actually matches.

**`.github/skills/new-class/SKILL.md`**

```markdown
---
name: new-class
description: Use when adding a new class to this project.
---

# Adding a new class

1. Make all fields `private`. Use `final` for anything that shouldn't change
   after construction.
2. Validate every constructor parameter before assigning it. Throw
   IllegalArgumentException with a clear message if something is invalid.
3. Add getters. Only add a setter if the field genuinely needs to change later.
4. Override `toString()` so the object prints readably.
5. Add a few lines to ShopDemo that create the object and print it.
```

Now ask Copilot to add a `Supplier` class with a name and a contact email. Without the skill it invents its own shape. With it, you get private final fields, a validating constructor, getters, `toString`, and demo lines in `ShopDemo` — the same way, every time.

The `description` is the trigger: Copilot reads descriptions to decide whether a skill is relevant. You can also run it directly by typing `/new-class` in chat.

| | Instructions | Skill |
|---|---|---|
| Shape | A list of rules | Numbered steps, in order |
| Loaded | Always, or when `applyTo` matches | Only when the task matches |
| Example | "Never catch generic Exception" | "To add a class: first this, then this" |

**The simple version:** instructions are the house rules on the wall. A skill is a recipe card you pull off the shelf when that job comes up. Add one when you notice yourself explaining the same procedure to Copilot more than once.

---

# Part 4: Agent Mode — Autonomous Development (35 min)

Everything so far has been you directing Copilot precisely. Agent Mode is different: you describe an outcome, and it works out the steps.

## What Agent Mode Does

Given a task, it:

1. Searches your project to understand the structure
2. Decides which files need creating or modifying
3. Makes changes across all of them
4. Runs commands if needed — compiling, running the program
5. Reads any errors and corrects itself
6. Presents everything for you to keep or undo

Steps 4 and 5 are what separate this from everything else. It doesn't just write code — it checks whether the code worked.

## Demo: Adding Refund Validation

**Switch the mode dropdown to Agent.** Then:

```
Add refund validation to the billing system.

Requirements:
- A refund cannot exceed the order total
- Total refunds across multiple partial refunds cannot exceed the order total
- Refunds can only be issued against orders that have been paid
- Throw IllegalStateException with a clear message when a refund is invalid
- Only mark an order REFUNDED when the full total has been refunded; use 
  PARTIALLY_REFUNDED otherwise
- Update ShopDemo to demonstrate a valid refund, a partial refund, and a 
  rejected over-refund

Follow the existing code style.
```

**Watch what it does.** It will work through the files one at a time, showing what it's changing as it goes.

**When it finishes:**

1. Review each changed file — click through the diffs
2. Check how it handled the "paid" requirement. `getPaymentStatus` returns a `String`, and the existing code uses string literals like `"PAID"`. Did Copilot keep using strings, or did it introduce an enum? **You didn't specify.** It made that decision for you.
3. Run `ShopDemo.java` and confirm all three refund scenarios behave correctly
4. Click **Keep** to accept, or **Undo** to revert

## The Point of That Exercise

You didn't tell it to add an enum, or a private helper, or whatever approach it chose. You described an outcome and it made design decisions on your behalf.

**That's the value and the risk in one sentence.** On code you understand, that's leverage — it did in two minutes what would have taken you twenty. On code you don't understand, you've just merged decisions you can't evaluate.

## Plan Mode

To see the reasoning before any code changes, switch the dropdown to **Plan**.

**Try the same prompt in Plan mode.** Instead of editing files, Copilot produces a written implementation plan — which files it intends to change and why. You approve, and only then does it execute.

For unfamiliar or high-risk code, this is the safer default.

## Activity: Debugging with Agent Mode (15 min)

Create a new file called `StockReport.java`. It is riddled with bugs — compile errors, crashes, and quiet wrong answers. Don't read it closely and don't try to fix anything. Hand it straight to the agent.

```java
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

public class StockReport {

    private final List<Integer> dailySales;
    private final Map<String, Integer> unitsByProduct;
    private String reportTitle;

    public StockReport(String reportTitle) {
        this.dailySales = new ArrayList<>();
        this.unitsByProduct = new HashMap<>();
    }

    public void recordSale(String product, int units) {
        dailySales.add(units)
        unitsByProduct.add(product, units);
    }

    public int totalUnits() {
        int total = 0;
        for (int i = 0; i < dailySales.length(); i++) {
            total += dailySales.get(i);
        }
        return total;
    }

    public double averageSales() {
        int total = 0;
        for (int i = 0; i < dailySales.size() - 1; i++) {
            total += dailySales.get(i);
        }
        return total / dailySales.size();
    }

    public int highestDay() {
        int highest = 0;
        for (int i = 1; i < dailySales.size(); i++) {
            if (dailySales.get(i) > highest) {
                highest = dailySales.get(i);
            }
        }
        return highest;
    }

    public int lowestDay() {
        int lowest = dailySales.get(0);
        for (int i = 0; i < dailySales.size(); i++) {
            if (dailySales.get(i) > lowest) {
                lowest = dailySales.get(i);
            }
        }
        return lowest;
    }

    public int daysAboveAverage() {
        int count = 0;
        for (int i = 0; i < dailySales.size(); i++) {
            if (dailySales.get(i) >= averageSales()) {
                count = 1;
            }
        }
        return count;
    }

    public double percentageOfTarget(int target) {
        return (dailySales.size() / target) * 100;
    }

    public boolean isTopProduct(String product) {
        String best = null;
        int bestUnits = 0;
        for (Map.Entry<String, Integer> entry : unitsByProduct.entrySet()) {
            if (entry.getValue() > bestUnits) {
                bestUnits = entry.getValue();
                best = entry.getKey();
            }
        }
        return product == best;
    }

    public String summary() {
        return reportTitle.toUpperCase() + ": " + totalUnits() + " units sold";
    }

    public static void main(String[] args) {
        StockReport report = new StockReport("Q1 Sales");

        report.recordSale("Keyboard", 10);
        report.recordSale("Cable", 14);
        report.recordSale("Keyboard", 6);
        report.recordSale("Monitor", 10);

        System.out.println(report.summary());
        System.out.println("Total units:        " + report.totalUnits());
        System.out.println("Average:            " + report.averageSales());
        System.out.println("Highest day:        " + report.highestDay());
        System.out.println("Lowest day:         " + report.lowestDay());
        System.out.println("Days above average: " + report.daysAboveAverage());
        System.out.println("Percent of target:  " + report.percentageOfTarget(50));
        System.out.println("Keyboard is top?    " + report.isTopProduct("Keyboard"));

        StockReport emptyReport = new StockReport("Empty");
        System.out.println("Lowest day:         " + emptyReport.lowestDay());
    }
}
```

### Step 1: Let the agent debug it

In **Agent** mode:

```
StockReport.java is broken. Compile it, run it, and fix whatever stops it
from working. Keep going until it runs cleanly.
```

Watch the chat panel. This takes several rounds, and you can see every one of them:

1. Compiles → `';' expected`. Fixes it.
2. Compiles → two more errors. `Map` has no `add()`, `List` has no `length()`. Fixes both.
3. Compiles clean. Runs → `NullPointerException`, because `reportTitle` is never assigned in the constructor. Fixes it.
4. Runs → `IndexOutOfBoundsException` from `lowestDay()` on an empty report. Adds a guard.
5. Runs clean. Reports it's done.

Click the arrow next to any command block to see the actual `javac` and `java` calls.

**Nobody told it what any of those were.** It found each one by running the code and reading what came back.

### Step 2: Now check the numbers

The agent says it's finished. The program runs. Every one of those fixes was correct.

**Here is what the output should be.** Work it out yourself from the sales data — 10, 14, 6 and 10, across Keyboard, Cable, Keyboard, Monitor:

```
Q1 SALES: 40 units sold
Total units:        40
Average:            10.0
Highest day:        14
Lowest day:         6
Days above average: 1
Percent of target:  80.0
Keyboard is top?    true
```

Compare that against what you actually got. Depending on your model, several of these will still be wrong:

| Line | Common wrong answer | The bug behind it |
|---|---|---|
| Average | `7.0` | Loop stops one short, **and** integer division |
| Lowest day | `14` | Comparison is `>` where it should be `<` |
| Percent of target | `0.0` | Uses day count instead of total units, **and** integer division |
| Keyboard is top? | `false` | `put()` overwrites instead of accumulating, **and** `==` instead of `.equals()` |

Not one of these threw an exception. Not one produced a compiler warning. The program exited cleanly with every one of them wrong.

Feed them back one at a time, with the expected value:

```
lowestDay() returns 14 for the values 10, 14, 6, 10. It should return 6.
Find and fix the cause.
```

### Step 3: The bug the test data is hiding

Look at `highestDay()` once more, even if it printed the right answer:

```java
int highest = 0;
for (int i = 1; i < dailySales.size(); i++) {
```

It starts at index **1**, so the first day is never considered. It also starts from `0`, so every value being negative would return `0`.

It printed `14` and looked fine — because the highest value happens to sit at index 1 in this data.

Add one line to `main` before the others:

```java
report.recordSale("Monitor", 99);
```

Now run it. The highest day is 99 and it still won't say so.

**The agent could only see what the run revealed.** The bug was there the whole time; the test data just never exposed it.

### Step 4: One decision it made for you

Check what it did about the empty report. Most likely you'll see:

```
Lowest day:         0
```

Nothing in your instructions said what an empty report should do. Returning `0` is a design decision it made silently — and it's the choice that hides the problem, because a caller now can't tell "no sales recorded" from "sales were zero."

```
Why did you return 0 for an empty report rather than throwing? Which would
you choose for a reporting class, and what breaks with each?
```

### What That Showed You

| Signal the agent had | What it caught |
|---|---|
| Compiler errors | The semicolon, `Map.add`, `List.length` |
| Runtime exceptions | The null title, the empty-list crash |
| Exit code 0 | Nothing. It only means "it ran" |

Everything in the first two rows announced itself. Something handed the agent the evidence, and it dealt with each one quickly and correctly — that part is genuinely impressive, and it would have taken you a lot longer by hand.

Everything that stayed broken had one thing in common: **it produced a plausible number and no complaint from anything.** The compiler was satisfied. The runtime was satisfied. The agent was satisfied.

The only reason you found them is that you knew what the answers should be.

**A test would have closed that gap.** One assertion that `averageSales()` returns `10.0` turns a silent wrong answer into a failing signal — and the agent iterates on failing signals all day. That's what tests are really for here: not just catching your mistakes, but giving an autonomous agent something to tell it whether it's finished or merely quiet.

**And the habit worth keeping:** when you hand Copilot a logic bug, never ask *"is there a bug here?"* Tell it what you expected and what you got.

---

# Part 5: Custom Agents (5 min)

If there's a kind of request you make constantly, you shouldn't be retyping the instructions every time.

A **custom agent** is a saved, named mode with its own instructions — it appears in the mode dropdown alongside Ask, Agent, and Plan.

**Create `.github/agents/reviewer.agent.md`:**

```markdown
---
description: Reviews Java code for correctness and design problems without making edits.
name: Reviewer
---

# Review instructions

You are a senior Java engineer reviewing code before it reaches production.

Identify correctness bugs, unhandled edge cases, and design problems. For 
each issue, describe the specific scenario where it causes a failure and 
rate its severity as HIGH, MEDIUM, or LOW.

Pay particular attention to:
- Exception handling that hides failures rather than surfacing them
- Arithmetic on monetary values
- Mutable state exposed through getters
- Boundary conditions in comparisons

Do not edit any files. Report findings only.
```

**Now open the mode dropdown.** "Reviewer" appears as an option. Select it, open any class, and simply say:

```
Review this
```

You get the same structured review without retyping the instructions.

> **Why this matters in a team:** a custom agent committed to the repository means every engineer reviews against the same standards. It turns one person's review checklist into something the whole team applies automatically.

---

# Wrap-Up

## What to Take Away

**The mode matters.** Ask, Agent, and Plan produce very different behaviour from the same prompt. Know which one you're in.

**Context is the constraint.** Copilot will find source files on its own, but it can't reach your errors, your terminal output, or a screenshot. Attach those. Check the "Used N references" line to see what it actually read.

**Specificity beats politeness.** A reasonable-sounding vague request produces a rewrite you have to audit. Stating the action, target, constraint, and standard produces exactly what you asked for.

**Fundamentals are what let you prompt well.** The precise prompt in Part 3 required knowing what an off-by-one error is, why catching generic `Exception` is dangerous, and what truncation does to money. Copilot didn't supply that knowledge. You did.

**Agent Mode makes design decisions.** It resolves ambiguity by choosing an approach. On familiar ground that's leverage; on unfamiliar ground it's a liability.

**A clean run is not a correct run.** The compiler and the runtime catch bugs that announce themselves. Whether anything catches a wrong-but-plausible number depends on the model that day — unless you wrote a test, or you knew the answer yourself.

## The Honest Summary

Copilot moved from autocomplete to autonomous agent in about three years. The features shift constantly — commands get renamed, panels get merged, modes disappear. You saw several examples of that in this lesson.

What doesn't change is the underlying relationship: **the better you understand the code, the more value you get from the tool.** Engineers who understand systems deeply use this to move considerably faster. Engineers who don't use it to generate code they can't evaluate.

The tool amplifies whichever one you are.

---



END