
### `package.json` setup
```bash
npm init -y
npm install @playwright/test
npx playwright install
npx playwright test --init
```

---

### `playwright.config.ts`
```ts
import { defineConfig } from '@playwright/test';

export default defineConfig({
  reporter: [['html'], ['list'], ['json', { outputFile: 'results.json' }]],
  retries: 2,
  timeout: 30000,
  use: {
    baseURL: 'https://jsonplaceholder.typicode.com',
  },
});
```

---

### `tests/ui/fileUpload.spec.ts`
```ts
import { test, expect } from '@playwright/test';

test.beforeEach(async ({ page }) => {
  await page.goto('http://localhost:3000'); // Update to your app URL
});

test('Page loads and URL is correct', async ({ page }) => {
  await expect(page).toHaveURL(/localhost/);
});

test('Elements are present', async ({ page }) => {
  await expect(page.locator('input[type="file"]')).toBeVisible();
  await expect(page.locator('button[type="submit"]')).toBeEnabled();
});

test('File upload functionality', async ({ page }) => {
  const fileInput = page.locator('input[type="file"]');
  await fileInput.setInputFiles('tests/fixtures/sample.pdf');
  await expect(page.locator('.file-name')).toContainText('sample.pdf');
});

test('Submit without file shows error', async ({ page }) => {
  await page.click('button[type="submit"]');
  await expect(page.locator('.error')).toBeVisible();
});

test('Success message after upload', async ({ page }) => {
  await page.locator('input[type="file"]').setInputFiles('tests/fixtures/sample.pdf');
  await page.click('button[type="submit"]');
  await expect(page.locator('.success')).toBeVisible();
});

test('Accessibility checks', async ({ page }) => {
  await expect(page.locator('label[for="file-upload"]')).toBeVisible();
  await expect(page.locator('button[type="submit"]')).toHaveAttribute('aria-label', /submit/i);
});
```

---

### Screenshot on Failure (add to tests)
```ts
test('example', async ({ page }) => {
  // test steps
}).afterEach(async ({ page }, testInfo) => {
  if (testInfo.status !== testInfo.expectedStatus) {
    await page.screenshot({ path: `screenshots/${testInfo.title}.png` });
  }
});
```

---

### `tests/api/posts.spec.ts`
```ts
import { test, expect } from '@playwright/test';

test.describe('Posts API tests', () => {

  test('GET /posts returns list', async ({ request }) => {
    const response = await request.get('/posts');
    expect(response.status()).toBe(200);
    const data = await response.json();
    expect(Array.isArray(data)).toBeTruthy();
    expect(data.length).toBeGreaterThan(0);
  });

  test('GET /posts/:id returns correct post', async ({ request }) => {
    const response = await request.get('/posts/1');
    expect(response.status()).toBe(200);
    const data = await response.json();
    expect(data.id).toBe(1);
  });

  test('POST /posts creates a new post', async ({ request }) => {
    const newPost = { title: 'foo', body: 'bar', userId: 1 };
    const response = await request.post('/posts', { data: newPost });
    expect(response.status()).toBe(201);
    const responseData = await response.json();
    expect(responseData).toMatchObject(newPost);
  });

  test('PUT /posts/:id updates a post', async ({ request }) => {
    const updated = { id: 1, title: 'updated', body: 'updated body', userId: 1 };
    const response = await request.put('/posts/1', { data: updated });
    expect(response.status()).toBe(200);
    const responseData = await response.json();
    expect(responseData).toMatchObject(updated);
  });

  test('DELETE /posts/:id deletes a post', async ({ request }) => {
    const response = await request.delete('/posts/1');
    expect([200, 204]).toContain(response.status());
  });

  test('GET /posts/:id for invalid id returns 404', async ({ request }) => {
    const response = await request.get('/posts/999999');
    expect(response.status()).toBe(404);
  });
});
