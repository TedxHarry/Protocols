# 📝 SCIM Series Master Prompt

This is the **master prompt** for generating the SCIM Learning Series in GitHub Markdown files.  
Use this prompt whenever you want to generate a section file. Replace the **section number and topic title** with the one you need.  

---

## 🎯 Core Goals
- Teach SCIM **step by step, concept by concept, topic by topic**.  
- Learners should **not feel overwhelmed** but also **not miss any important concept**.  
- Each article should be **clear, well-explained, and professional**.  
- Style should be like a LinkedIn article: **engaging, clean, with emojis and short sections**.  
- Always add **Back / Next navigation links** at the bottom of each file.  
- Never use em dashes.  
- No analogies — focus on real meaning.  
- Self-check at the end of each file (2–3 questions) tied directly to that article’s content.  

---

## 📚 Series Roadmap (Finalized)

### 🏆 Section 1 | SCIM Foundations  
1.01 What is SCIM?  
1.02 SCIM Architecture  
1.03 Core Objects  
1.04 SCIM Endpoints  
1.05 Lifecycle Operations  
1.06 Filtering Basics  
1.07 Common Errors & Pitfalls  
1.08 Extensions  

### 🏆 Section 2 | SCIM in Practice  
- Attribute Mapping Basics  
- Multi-Valued Attributes  
- PATCH vs PUT  
- Pagination & Sorting  
- Bulk Operations  
- Error Handling & Monitoring  

### 🏆 Section 3 | SCIM Integrations (Real Tools)  
- Okta SCIM  
- Entra ID SCIM  
- SailPoint SCIM  
- ServiceNow SCIM  
- Google Workspace SCIM  

### 🏆 Section 4 | Best Practices & Readiness  
- Common Pitfalls and Fixes  
- Debugging SCIM Calls  
- Monitoring Provisioning Jobs  
- Compliance and Audit  
- Production Readiness Checklist  

---

## 🏗️ File Format (Dynamic, Adapts per Topic)

Each file should include (adapt dynamically per topic — don’t force if irrelevant):  

1. **Title with Section Number and Topic**  
   Example:  
   `# 🏆 Section 1.05 | SCIM Foundations | “Lifecycle Operations”`  

2. **Intro**  
   Short, clear explanation of why the topic matters.  

3. **Core Concepts**  
   Step-by-step explanation of the concept in beginner-friendly terms.  

4. **Examples (Optional)**  
   Minimal but relevant JSON/HTTP snippets when helpful.  

5. **Why It Matters**  
   Governance, security, operational importance.  

6. **Real-World Example**  
   Direct workplace-relevant scenario.  

7. **Common Pitfalls (Optional)**  
   Mistakes to avoid, explained clearly.  

8. **Self-Check**  
   2–3 questions for readers to reflect on.  

9. **Final Takeaway**  
   A one-paragraph wrap-up that reinforces the concept.  

10. **Navigation**  
   Example:  
   ```
   ## 🔗 Navigation  
   👉 Back: [1.04 SCIM Endpoints](1.04-scim-endpoints.md)  
   👉 Next: [1.06 Filtering Basics](1.06-filtering-and-querying.md)  
   ```  

---

## 🛠️ Output Rules
- Output must be a **GitHub-ready Markdown file** (`.md`) with full, detailed content.  
- Do not compress, trim, or shorten content.  
- Content must be detailed enough to give learners **real knowledge and confidence**.  

---

👉 Use this master prompt whenever you want to generate a specific section file in the SCIM series.  
