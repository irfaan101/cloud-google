# 🧹 Smart Mailbox Sanitizer — AI-Driven Gmail Management System

**Developed By:**  
Irfaan Mansoori, B.Tech CSE  

**Capstone / Major Project Track:**  
AI & Automation Systems  
*(Also suitable for Cloud Computing / Machine Learning / Software Development)*  

---

## 📌 Problem Statement

Modern email inboxes are cluttered with:  

- Spam messages  
- Promotional ads  
- Endless newsletters  
- Social notifications  

Manually sorting emails is slow, repetitive, and inefficient. Users need a **smart, automated system** to clean and organize their Gmail inbox **without manual effort**.

---

## 🎯 Project Objectives

- Automatically categorize emails using AI:  
  `Spam / Newsletter / Promotions / Social / Important`  
- Delete spam and irrelevant promotions  
- Auto-unsubscribe from unwanted newsletters  
- Archive low-priority social emails  
- Secure important emails  
- Execute automatically every day using cloud-based scheduling

---

## 💡 Proposed Solution

**Smart Mailbox Sanitizer** is an **AI-powered Gmail automation tool** that:  

- Fetches emails from Gmail via the Gmail API  
- Classifies emails using **Google Gemini API** (LLM-based classifier)  
- Automatically applies actions based on category: delete, archive, or preserve  
- Runs **daily** using **Kaggle Notebook Scheduler**  

This ensures a clean, organized, and stress-free Gmail inbox continuously.

---

## 🛠 Tech Stack

- Python  
- Google Gemini API (LLM)  
- Gmail API (OAuth 2.0)  
- Kaggle Notebooks & Kaggle Secrets  
- REST APIs  

---

## 🏗 System Architecture

The **System Architecture** of the Smart Mailbox Sanitizer shows how all components interact to automate Gmail management.

### Workflow Overview

1. **User Gmail** – The source of emails (inbox, spam, social, promotions).  
2. **Email Extraction** – Emails are fetched from Gmail using the **Gmail REST API**.  
3. **Gemini AI Classifier** – Each email is analyzed and categorized into:  
   - Spam  
   - Promotion  
   - Newsletter  
   - Social  
   - Important  
4. **Automation Engine** – Performs actions based on email category:  
   - **Delete Spam:** Removes unwanted emails permanently.  
   - **Remove Promotions:** Deletes irrelevant promotional emails.  
   - **Auto-Unsubscribe Newsletters:** Extracts unsubscribe links and unsubscribes automatically.  
   - **Archive Social Emails:** Moves low-priority social notifications out of the inbox.  
   - **Preserve Important Emails:** Ensures crucial emails remain untouched.  
5. **Scheduler (Kaggle Notebook)** – Executes the entire process automatically every day.

### Architecture Diagram

User Gmail

   ↓ (via Gmail API)

Email Extraction

   ↓

Gemini AI Classifier

   ↓

Automation Engine
 ├─ Delete Spam

 ├─ Remove Promotional Emails

 ├─ Auto-Unsubscribe Newsletters

 ├─ Archive Social Emails

 └─ Preserve Important Emails

   ↓

Kaggle Notebook Scheduler → Runs Daily Automatically
