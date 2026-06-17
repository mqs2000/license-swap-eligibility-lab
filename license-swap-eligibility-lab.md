# Lab: Building a License Swap Eligibility Agent with IBM watsonx Orchestrate

## Overview

In this hands-on lab, you will build a no-code AI agent using **IBM watsonx Orchestrate** that helps users check whether they qualify to swap a foreign driving license for a UAE driving license — without sitting a driving test.

The agent guides the user through the eligibility process step by step:

1. Collects the user's driving license via file upload
2. Automatically extracts details from the license using a Document Extractor
3. Applies deterministic eligibility logic via a Branch condition
4. Returns a personalised confirmation message (eligible) or a branch-visit message (not eligible)

---

## Prerequisites

- An IBM Cloud account you can get one here -> https://www.ibm.com/account/reg/us-en/signup?formid=urx-52753

---

## Step 1 — Create a watsonx Orchestrate Trial Instance

1. Go to the **IBM SaaS Console** [https://console.saas.ibm.com/dashboard/subscriptions] and navigate to **Instances**.
   > **Note:** You might be asked to set up multi-factor authentication. Choose email, wait for the code, and verify.

![IBM SaaS Console – Instances view](images/step1-instances.png)


2. Click **Create instance +** (top-right).

![Create instance button](images/step1-create-instance-btn.png)

3. Fill in / choose the instance details as follows:

![Instance details form](images/step1-instance-details.png)

4. Click **Create**.
5. Once the instance status shows **Running**, click **Launch** to open watsonx Orchestrate.

![Instance running – Launch button](images/step1-running-launch.png)

> **Note:** If you see "Your account is not provisioned yet", wait a few minutes and then try launching again.

---

## Step 2 — Create the Agent

1. From the watsonx Orchestrate home screen, click **Create your agent**.

![watsonx Orchestrate home – Create your agent](images/step2-create-agent.png)

2. In the dialog, select **Create from scratch**.

![Create from scratch dialog](images/step2-from-scratch.png)

3. Enter the following details:

   | Field | Value |
   |---|---|
   | **Name** | `License Swap Eligibility Agent` |
   | **Description** | See full description below |

   **Agent Description:**
   > You are a License Swap Eligibility Agent to help users check if they qualify to swap a foreign driving license for a UAE license without taking tests. Using the License Swap Eligibility Check tool, you guide users through each step of the eligibility process, starting with license upload and automated detail extraction, followed by eligibility checks using deterministic logic. The tool verifies that the issuing country is eligible, that the license is still valid, and that the applicant is at least 18. You handle human-in-the-loop activities like form submissions and confirmation messages, and generate personalized responses using AI prompts.

![](images/step2-agent-details.png)

4. Click **Create**.


---

## Step 3 — Configure the Agent Profile

After the agent is created, you will be on the **Profile** tab.

### Welcome Message

Update the default welcome message to:

```
Hello, welcome to License Swap Agent
```

### Quick Start Prompts

1. Click **Add prompt +**.
2. Enter: `Check License swap eligibility.`
3. Remove all the other prompts.

![Quick start prompts configured](images/step3-quick-prompts.png)

---

## Step 4 — Configure Agent Behavior

1. Click **Behavior** in the left sidebar.
2. In the **Instructions** field, enter:

   > When the user requests to swap a foreign license for a UAE license, call the tool "License Swap Eligibility Check" and do not generate or display any additional output. The tool is fully responsible for handling the process and presenting the final response to the user. Your role is strictly to delegate the task to the tool. Do not repeat, summarize, or echo the tool's output, as it already includes the necessary user-facing messages.

![Behavior instructions configured](images/step4-behavior.png)

---

## Step 5 — Create the Workflow Tool

### 5.1 Open the Toolset and Add a Tool

1. Click **Toolset** in the left sidebar.
2. Click **Add tool +**.

![Add tool – Agentic workflow option](images/step5-add-tool.png)

3. In the "Add a tool" dialog, select **Agentic workflow** under *Create new*.

![Add tool – Agentic workflow option](images/step5-add-agentic-workflow.png)

4. When prompted to name the workflow, enter: `License Swap Eligibility Check`
5. Click **Start building**.

---

### 5.2 Workflow Step 1 — Display a Welcome Message

You are now in the agentic workflow canvas.

1. Click **Add your first step +** (or the `+` button between Start and End nodes).
2. Select **Present to user → Message**.

![Agentic workflow canvas](images/step5-canvas.png)

4. Under **Output message**, enter:

   ```
   To get started please upload the driving license.
   ```

![Welcome message node configured](images/step5-welcome-message.png)

---

### 5.3 Workflow Step 2 — Create the File Upload Node

1. Click the **+** button below the Message node.

![Plus button below Message node](images/step5-plus-below-message.png)

2. Select **Collect from user → File upload**.

![File upload node added](images/step5-file-upload.png)

3. This adds a **File upload** step that lets the user attach their license image or PDF.

---

### 5.4 Workflow Step 3 — Extract License Details

1. Click the **+** button **below** the User activity block (outside the green dashed box, on the main canvas).
2. Select **Add a flow activity → Document extractor**.

![Document extractor selected](images/step5-doc-extractor.png)

3. When asked to select a document format, choose **Structured** (since a driving license has a consistent layout).

![Structured format selected](images/step5-structured.png)

4. Click **Define schema** in the Document extractor configuration panel.
5. In the schema selection dialog, choose the predefined schema: **Driver license**.

![Driver license schema selected](images/step5-driver-license-schema.png)

   This schema includes the following fields that will be automatically extracted. Remove all fields that are not listed below:

   | Field | Type |
   |---|---|
   | License number | string |
   | Full name | string |
   | Date of birth | date |
   | Nationality | string |
   | Date of issue | date |
   | Date of expiration | date |

6. Upload a sample license image on the right panel to test extraction before proceeding. Use the image `Training.png` provided in the repo.

![Sample license extraction test](images/step5-extraction-test.png)

7. Click **Create**.

---

### 5.5 Workflow Step 4 — Add a Branch (Eligibility Check)

1. Click the **+** button below the Document extractor node.
2. Select **Add a flow control → Branch**.

![Workflow canvas overview](images/step5-Branch.png)

   A **Branch 1** diamond node appears with two default paths:
   - **Path 1** (if condition)
   - **Path 2** (default / else)

---

### 5.6 Configure Path 1 — Ineligible Nationalities

Path 1 will route users whose nationality does **not** qualify for a license swap.

1. Click on **Branch 1**, then click **Edit condition** next to **Path 1**.

![Workflow canvas overview](images/step5-Edit-condition.png)

3. Click **+**.
4. 
   - Flow Variables → **Document extractor** → `nationality`

![Workflow canvas overview](images/step5-Edit-condition-details.png)
     
   - Operator: `==`
   - Value: `Cameroonian`

4. Click **+** again, change the logical operator to **or**, and add:
   - `nationality == Indian`

![Workflow canvas overview](images/step5-Edit-condition-or.png)

5. Repeat to add:
   - `nationality == Pakistani`
   - `nationality == Nigerian`

---

### 5.7 Add a Generative Prompt for the Not-Eligible Path (Path 1)

Path 1 uses a **Generative prompt** to produce a personalised not-eligible response.

1. In Path 1, click the **+** icon to add a step.
2. Select **Add a flow activity → Generative prompt**.
3. Rename it to `Not-Eligible prompt` (click the node name to edit).
4. In the configuration panel:

   **System prompt:**
   ```
   You are an assistant that generates the final confirmation message for a license swap application. Your task is to take the provided user details (license number, full name, date of birth, nationality, date of issue, and date of expiration) and produce a complete, ready-to-send message. Do not provide instructions or code, only return the final text output.
   ```

5. Add the following **inputs** by clicking **Add +** → **String** for each:
   - `full_name` — map to `Document extractor → full_name`
   - `nationality` — map to `Document extractor → nationality`

![Workflow canvas overview](images/step5-Not-Eligible-Path.png)

6. **User prompt:**

   ```
   Generate a message using the following details:
Full Name: {self.input.full_name}
Nationality: {self.input.nationality}


The message should:
- Start with a greeting addressing the customer by their full name.
- Politely inform the customer that, based on their details, they are not eligible for a license swap at this time.
- Advise the customer to contact or visit the center for further assistance and to discuss available options.
- Avoid stating a specific reason for ineligibility.
- Do NOT include a reference number.
- Do NOT confirm any request as submitted.
- Maintain a respectful and supportive tone.
- End with the company name: Road and Transportation Authority.
   ```

7. Click **Generate Preview** to verify the output looks correct.
8. Close the Generative prompt panel.

---

### 5.8 Configure Path 2 (Default) — Eligible Path

Path 2 (the default/else path) handles eligible nationalities. Here you will add a **Generative prompt** to compose a personalised confirmation message.

1. In Path 2, click **+** to add a step.
2. Select **Add a flow activity → Generative prompt**.
3. Rename it to `Eligible prompt` (click the node name to edit).
4. In the Generative prompt configuration:

   **System prompt:**
   ```
   You are an assistant that generates the final confirmation message for a license swap application. Your task is to take the provided user details (license number, full name, date of birth, nationality, date of issue, and date of expiration) and produce a complete, ready-to-send message. Do not provide instructions or code, only return the final text output.
   ```

5. Add the following **inputs** by clicking **Add +** → **String** for:
   - `full_name` — map to `Document extractor → full_name`
   - `nationality` — map to `Document extractor → nationality`

![Workflow canvas overview](images/step5-Eligible-Path-String.png)

   **Add +** → **date** for:
   - `issue_date` — map to `Document extractor → date_of_issue`
   - `expiry_date` — map to `Document extractor → date_of_expiration`
   - `birth_date` — map to `Document extractor → date_of_birth`

6. **User prompt:**

   ```
   Generate a confirmation message using the following details:

Full Name: {self.input.full_name}

Nationality: {self.input.nationality}

Issue Date: {self.input.issue_date}

Expiry Date: {self.input.expiry_date}

Birth Date: {self.input.birth_date}

The message should:

- Start with a greeting and thanking the full name of the customer
- Confirm that the User is eligible for license swap.
- Confirm that the license swap request was submitted successfully.
- Include a random 8-digit reference number.
- State that one of our agents will contact the customer for the next steps.
- End with the company name: Road and Transportation Authority.
   ```

7. Click **Generate Preview** to verify the output looks correct.
8. Close the Generative prompt panel.

---

### 5.9 Map Both Generative Prompts to the Output Node

Both paths feed into a single **Output** node. You need to map each generative prompt's output to a named output variable.

1. Click the **Output** node on the canvas.
2. Click **Add** → **String**:
   - Name: Eligible.
   - Click **Add**
4. Repeat the same thing for Not-Eligible output Click **Add** → **String**:
   - Name: Not-Eligible.
   - Click **Add** 
5. Click **Edit data mapping**

![Map data for End – output mapping](images/step6-edit-map.png)

7. Click on **{x}** for Eligible
   - From Flow Varibales choose **Eligbile** → **Value**.

   ![Map data for End – output mapping](images/step6-map-x.png)
   
8. Do the same for Not-Eligible Click on **{x}**.
   - From Flow Varibales choose **Not-Eligible** → **Value**.

---

### 5.10 Exit the Workflow

1. Click **Done** (top-right) to return to the agent configuration.

---

## Step 7 — Test the Agent

1. In the right-hand panel, click **Talk to agent**.
2. Click the quick-start prompt: **Check License swap eligibility.**
3. The agent should automatically invoke the workflow tool and display:

   > *To get started please upload the driving license.*

4. Upload a sample driving license image (PNG, JPG, PDF, etc.).
5. The document extractor will parse the license and extract fields automatically.
6. Depending on the nationality on the license:
   - **Eligible nationality** → You will receive a personalised confirmation message with a reference number.
   - **Ineligible nationality** → You will see: *This country is not eligible for a license swap. Please visit the branch for further assistance.*

---

## Workflow Summary

```
[Start]
   │
   ▼
[User Activity 1]
   ├── Message 1: "To get started please upload the driving license."
   └── File Upload 1
   │
   ▼
[Document Extractor]  ← extracts: license_number, full_name, date_of_birth,
   │                              nationality, date_of_issue, date_of_expiration
   ▼
[Branch 1]
   ├── Path 1 (if nationality is ineligible)
   │       └── [Generative Prompt neg] ← generates personalised not-eligible message
   │
   └── Path 2 / default (eligible)
           └── [Generative Prompt] ← generates personalised confirmation
   │
   ▼
[End]
   ├── result     ← Generative Prompt.value
   └── result_neg ← Generative Prompt neg.value
```

---

## Key Concepts Used

| Component | Purpose |
|---|---|
| **Agentic Workflow** | Low-code tool that automates the eligibility check process end-to-end |
| **Document Extractor** | Extracts structured fields from the uploaded license image using AI |
| **Branch** | Routes the flow based on deterministic conditions (nationality) |
| **Generative Prompt** | Uses an LLM to compose a personalised confirmation message |
| **Behavior Instructions** | Tells the agent to delegate fully to the tool without adding extra output |

---

## Conclusion

You have successfully built a no-code License Swap Eligibility Agent using IBM watsonx Orchestrate. The agent:

- Accepts a driving license upload from the user
- Automatically extracts key details using a document extractor
- Applies deterministic eligibility logic via a branch
- Returns a personalised AI-generated confirmation (for eligible users) or a branch-visit message (for ineligible users)

This pattern — combining document extraction, conditional branching, and generative AI — can be extended to many other document-driven eligibility workflows.
