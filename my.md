### All Submissions:

- [x] Have you followed the [WooCommerce Contributing guideline](https://github.com/woocommerce/woocommerce/blob/trunk/.github/CONTRIBUTING.md)?
- [x] Does your code follow the [WordPress' coding standards](https://make.wordpress.org/core/handbook/best-practices/coding-standards/)?
- [x] Have you checked to ensure there aren't other open [Pull Requests](https://github.com/woocommerce/woocommerce/pulls) for the same update/change?

### Changes proposed in this Pull Request:

Closes:

- https://github.com/woocommerce/woocommerce-quality/issues/336
- https://github.com/woocommerce/woocommerce-quality/issues/337
- https://github.com/woocommerce/woocommerce-quality/issues/338
- https://github.com/woocommerce/woocommerce-quality/issues/339

This PR contains the following changes:

1. [`Smoke test release`](https://github.com/woocommerce/woocommerce/actions/workflows/smoke-test-release.yml) workflow now uses the Playwright tests, both API and E2E.
1. Run the `Smoke test release` workflow:
   - automatically when a new release is published, including pre-releases.
   - manually, by running it from the [`Smoke test release`](https://github.com/woocommerce/woocommerce/actions/workflows/smoke-test-release.yml) page.
1. The workflow runs the API & E2E tests in the following environments:
   | WP & PHP version | Site |
   | ---------------- | ---- |
   | WP Latest, PHP 8.0 | Release Smoke Test site in Bluehost |
   | WP Latest-1 | `wp-env` localhost |
   | WP Latest-2 | `wp-env` localhost |
   | PHP 7.4 | `wp-env` localhost |
   | PHP 8.1 | `wp-env` localhost |
1. Publish Allure report for each test on [WooCommerce Test Reports > Smoke Tests on Releases](https://woocommerce.github.io/woocommerce-test-reports/release/) page.
1. Address several more flakiness discovered when running the E2E tests against the Release Smoke Test Site.
1. The `UPDATE_WC` environment variable now accepts the WooCommerce version you want to run the tests against.

For now, this PR **does not** include running of tests with top plugins (WooCommerce Payments, WooCommerce Subscriptions, etc.).

### How to test the changes in this Pull Request:

#### Scenario 1: Test using a valid WooCommerce release version as input

1. Go to [`Smoke test release`](https://github.com/woocommerce/woocommerce/actions/workflows/smoke-test-release.yml).
1. Trigger a new run using these steps:
   1. Click _Run workflow_.
   1. Select this branch (`e2e/playwright-release`).
   1. In the _WooCommerce Release Tag_ field enter a valid WooCommerce release version like `7.4.0-beta.1` or `7.0.0`. Do not use `7.2.0-rc.2` because it doesn't have a `woocommerce.zip` asset. Also, choose a version that's not yet listed in [WooCommerce Test Reports > Smoke Tests on Releases](https://woocommerce.github.io/woocommerce-test-reports/release/).
   1. Start the run.
1. Go to the workflow run you started.
1. Wait for the _Test WooCommerce update_ job to finish. It runs the `update-woocommerce.spec.js` E2E test against the Release Smoke test site to see update failures early on.
<!-- Insert image here -->
1. Go to [WooCommerce Test Reports > Smoke Tests on Releases](https://woocommerce.github.io/woocommerce-test-reports/release/). The WooCommerce version number you entered earlier should appear on the list. If not, wait for 1-2 more minutes to give GitHub Pages more time to upload the new report from _Test WooCommerce update_ job.
1. Click on the version number to expand it.
1. In the _WP Latest_ row:
   1. Verify the "E2E" column using these steps:
      1. You should see "E2E :white_check_mark:" which means that `update-woocommerce.spec.js` passed on the Release smoke test site.
      1. Click on the "E2E" link. You should see the Allure report containing the passing `update-woocommerce.spec.js` test.
      1. It's fine if you see `can run the database update` marked as "skipped" because the WC database version in our Release Smoke Test site is already the latest.
   1. Verify the "API" column using these steps:
      1. You should see the "API" text to be grayed out for now because the API tests haven't finished yet. Same with the other "E2E" and "API" tests of other environments in the list.
      <!-- Insert image here -->
1. We will now verify if the previous 2 versions of WordPress were correctly obtained. Go back to the `Smoke test release` run.
1. Wait for the _Get WP L-1 & L-2 version numbers_ job to finish, if it hasn't yet.
1. Click _Get WP L-1 & L-2 version numbers_ to see its steps.
1. Click the step _Get version numbers_. It should print the correct L-1 and L-2 versions obtained from the [WordPress.org API version check endpoint](https://api.wordpress.org/core/version-check/1.7/). Verify that they're correct.
<!-- Insert image here -->
1. Now, verify if the obtained PHP versions were correct. Click on any of the _Test against PHP X.X_ jobs.
1. Expand the _Verify PHP version_ step. Verify that the `PHP version found in WP Env environment` is correct when compared to the `Expected PHP version` as shown below.
<!-- Insert image here -->
1. To verify that the tests ran on the WP L-1 and L-2 versions, click on any of the _Test against WP Latest-X_ jobs.
1. Look for the step _Verify environment details_ and click to expand it. This step basically runs shell commands that print certain environment details like the WordPress version installed.
1. Verify that the printed WP version is correct.
<!-- Insert image here -->
1. Still under the _Verify environment details_ step, verify that the installed WooCommerce version is the one that you selected earlier.
<!-- Insert image here -->
1. Wait for all the jobs to finish.
1. Go back to [WooCommerce Test Reports > Smoke Tests on Releases](https://woocommerce.github.io/woocommerce-test-reports/release/) and expand the WooCommerce version you chose.
1. You should see the table filled up now with status icons, except for `PHP 8.0` because it's already covered by `WP Latest` which ran against the Release Smoke Test site, which has PHP 8.0. Refer to the "Legend" section to see what each icon means.
<!-- Insert image here -->
1. In the _"WP Latest"_ row, click on the "E2E" link again. You should now see the Allure report of all tests, including the `update-woocommerce.spec.js` test you saw earlier.
1. Click on the E2E and API links of other rows to see if you're taken to the correct Allure report.
1. In the Allure report, verify the correctness of the following details:
<!-- Insert image here -->

#### Scenario 2: Test using an invalid WooCommerce release version as input

1. Go to [`Smoke test release`](https://github.com/woocommerce/woocommerce/actions/workflows/smoke-test-release.yml).
1. Trigger a new run using these steps:
   1. Click _Run workflow_.
   1. Select this branch (`e2e/playwright-release`).
   1. In the _WooCommerce Release Tag_ field enter an invalid value like `7.6.0` or `7.3.0-beta.3`.
   1. Start the run.
1. Open the workflow run you just started.
1. After a few moments, verify that:
   1. The _"Get WooCommerce release tag"_ job fails
   1. All the other jobs did not execute.
   1. Under the _"Validate tag"_ step, the error `release not found` is printed.
   <!-- Insert image here -->

#### Scenario 3: Test using an valid WooCommerce release version, but does not have a `woocommerce.zip` release asset

1. Go to [`Smoke test release`](https://github.com/woocommerce/woocommerce/actions/workflows/smoke-test-release.yml).
1. Trigger a new run using these steps:
   1. Click _Run workflow_.
   1. Select this branch (`e2e/playwright-release`).
   1. In the _WooCommerce Release Tag_ field enter a valid release tag in the WooCommerce Core repo, but does not have a `woocommerce.zip` asset like `7.2.0-rc.2` or `wc-beta-tester-2.2.0`.
   1. Start the run.
1. Open the workflow run.
1. After a few moments, verify that:
   1. The _"Get WooCommerce release tag"_ job fails.
   1. All the other jobs did not execute.
   1. The _"Verify woocommerce.zip asset"_ step fails because there's no woocommerce.zip found
   <!-- Insert image here -->

### Other information:

- [x] Have you added an explanation of what your changes do and why you'd like us to include them?
- [x] Have you created a changelog file for each project being changed, ie `pnpm --filter=<project> changelog add`?

### FOR PR REVIEWER ONLY:

- [x] I have reviewed that everything is sanitized/escaped appropriately for any SQL or XSS injection possibilities. I made sure Linting is not ignored or disabled.
