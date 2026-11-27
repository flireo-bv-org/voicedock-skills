---
name: voicedock-whitelabel-partners
description: Work as a VoiceDock whitelabel partner on the assistants of your customers - find the customer's organisation, make sure every call runs in the right one, copy a proven assistant from your own organisation, and know what to do in the dashboard instead. Use when a partner, agency or reseller asks to set up, change or check an assistant for one of their customers, or when listOrganizations shows organisations with a partner_org_id.
---

# Working on your customers' assistants as a whitelabel partner

As owner or admin of a whitelabel partner organisation, your VoiceDock connection also reaches the organisations of your customers. Each customer is a separate organisation with its own assistants, numbers and calls. Most mistakes here are about the organisation, not the assistant: a change made in the wrong customer is a change a real business hears on its phone line.

## 1. Find the customer's organisation

Call `listOrganizations` first. Customer organisations have role `admin` and a `partner_org_id`, which is the id of your partner organisation.

| What you see | Meaning |
|---|---|
| Organisation with `partner_org_id` | A customer of the partner with that id. You can use every tool in it |
| Organisation without `partner_org_id`, role `owner` or `admin` | An organisation you are a member of yourself, often your own partner organisation |
| Two customers with the same or similar name | Ask the user which one. Never guess from the name |

If the customer the user names is not in the list, stop and say so. It may not be created yet, or it may belong to another partner.

## 2. Every call names the organisation

- Pass `organization_id` on every call, reads included. With more than one organisation a call without it is refused, and nothing happens.
- Each result names the organisation it came from under `organization`. Check that name before you act on what you read, and before every write.
- When the user switches to another customer mid-conversation, say which organisation you now work in.
- Never copy ids (assistant, number, tool) from one organisation into a call for another. Ids only exist in their own organisation.

## 3. Start from a proven assistant

Build and test an assistant in your own partner organisation first, then use it as the template for customers.

1. Read the template with `getAssistant` in your partner organisation.
2. Create the customer's assistant with `createAssistant` in the customer organisation, with the same configuration, and replace what is specific to the customer: business name, greeting, opening hours, addresses, transfer numbers, webhook URLs.
3. Check the result with `getAssistant` in the customer organisation and compare it field by field with what you intended.
4. Test it before a number points to it. See the `voicedock-test-and-review` skill.

When you update an assistant, send the complete configuration objects (`llm_config`, `stt_config`, `tts_config`): a partial object can remove settings you did not mean to change.

## 4. What stays in the dashboard

These steps are done on the **Whitelabel** page of the dashboard, not through the connector:

- creating a new customer (optionally from a template assistant),
- inviting the customer's users to their portal, resending a login link, revoking access,
- picking a phone number for a customer from your number stock,
- budgets, price plans and invoicing for your customers.

Tell the user which of these they need, then continue with the assistant work once it is done.

## 5. Before you finish

- Name the customer organisation and the assistant you changed, and what changed.
- If a number is linked, say which assistant it now reaches.
- Leave anything the customer will hear (greeting, prices, opening hours, legal text) for the user to confirm.

Docs: `https://doc.voicedock.ai/docs/integrations/mcp-server` (working with more than one organisation) and `https://doc.voicedock.ai/docs/platform/whitelabel`.
