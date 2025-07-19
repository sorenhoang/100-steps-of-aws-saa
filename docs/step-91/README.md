# Step 91: Exam Tips & Time Management

## 🎯 Objective
- [x] Master: **Exam Tips & Time Management**

## 📘 Notes

Of course. Let's shift gears from "what to know" to "how to pass." Here are essential tips for managing your time and tackling the questions on the SAA-C03 exam.

---
## 📝 Exam Format and Scoring

* **Questions:** 65 multiple-choice and multiple-response questions.
* **Time:** 130 minutes.
* **Passing Score:** The score is scaled, but you should aim for roughly 72% or higher.
* **Unscored Questions:** Know that about 15 of the 65 questions are unscored. They are used by AWS for research and do not affect your final score. You won't know which ones they are, so treat every question as if it's scored.

---
## ⏱️ Time Management Strategy

You have exactly **2 minutes per question** on average (130 minutes / 65 questions). Here's a practical way to manage your time:

1.  **Initial Pass (The Triage):** Go through all 65 questions.
    * **Easy Questions ( < 1 minute):** If you immediately know the answer, select it, and move on. Don't second-guess yourself. These questions buy you time.
    * **Medium Questions (1-2 minutes):** If you need to read the question carefully and think for a moment, use your allotted time. Eliminate wrong answers to increase your odds.
    * **Hard Questions ( > 2 minutes):** If a question is very long, involves many services, or you're unsure, make your best guess, **flag it for review**, and move on immediately. Do not get stuck on a single question.

2.  **Review Pass:** After you've answered all 65 questions, you should have 15-20 minutes left. Now, go back and review *only* the questions you flagged. Your brain will have been working on them in the background, and you can now look at them with a fresh perspective.

3.  **Final Check:** If you have any time left after the review pass, do a quick check for any obvious mistakes.

---
## 🤔 Question Strategy & Tips

How you read and interpret the questions is the most critical skill.

* **Identify Keywords and Qualifiers:** AWS questions are written very precisely. Look for the keywords that point you to a specific pillar or service.
    * **Cost:** "most cost-effective," "lowest cost," "reduce spending."
    * **Performance:** "lowest latency," "improve performance," "most efficient."
    * **Reliability:** "highly available," "fault-tolerant," "disaster recovery," "eliminate single points of failure."
    * **Operational Excellence:** "automate," "standardize," "deploy," "monitor."
    * **Security:** "encrypt," "least privilege," "protect," "audit."
    * Watch for qualifiers like **MOST**, **LEAST**, and **BEST**. These words mean that multiple answers might be technically possible, but only one is the optimal solution for the stated requirement.

* **Eliminate Incorrect Answers:** This is your most powerful tool. For any given question, you can almost always eliminate two of the four options immediately because they are technically incorrect or don't make sense in the context. This increases your odds of guessing correctly from 25% to 50%.

* **Differentiate Similar Services:** The exam will test your ability to choose between services that have similar functions.
    * **SQS vs. SNS:** Is it 1-to-1 decoupling (**SQS**) or 1-to-many fan-out (**SNS**)?
    * **NAT Gateway vs. Internet Gateway:** Does a private instance need outbound-only access (**NAT Gateway**), or does a public instance need two-way access (**Internet Gateway**)?
    * **EFS vs. FSx for Windows:** Is it a shared file system for **Linux** (**EFS**) or **Windows** (**FSx**)?
    * **GuardDuty vs. Inspector vs. Config:** Is it a **threat** (**GuardDuty**), a **vulnerability** (**Inspector**), or a **misconfiguration** (**Config**)?
    * **VPN vs. Direct Connect:** Is it a quick, low-cost connection (**VPN**) or a dedicated, high-performance connection (**Direct Connect**)?

* **Don't Fight the Question:** Read the question and answer exactly what it's asking. If it asks for the most cost-effective solution, choose the cheapest option that meets the requirements, even if another option is more performant or resilient. Don't add your own requirements to the scenario.
