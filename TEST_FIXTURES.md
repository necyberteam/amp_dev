# Test Fixtures for Cypress Tests

This document describes the test content created by the `amp_dev` module for use in Cypress tests.

## Overview

The `amp_dev.install` file creates test content when the module is installed (during database backup process). This provides consistent, reliable test data for Cypress tests without needing to create content during test runs.

## Test Announcements

All test announcements use the `ai` tag and have `Community` affiliation. They are created with different `field_choose_where_to_share_this` values to test visibility behavior.

| Title | Choose Where to Share | Purpose |
|-------|----------------------|---------|
| Test Announcement - Main Page Only | `on_the_announcements_page` | Should appear ONLY on `/announcements` |
| Test Announcement - Affinity Group Only | `on_your_affinity_group_page` | Should appear ONLY on `/affinity-groups/access-support` |
| Test Announcement - Both Locations | Both options above | Should appear on BOTH locations |
| Test Announcement - No Visibility | Empty array | Should NOT appear on any listing pages |
| Test Announcement - Email Only | `email_to_your_affinity_group` | Should only be sent via email |
| Test Announcement - Digest Only | `in_the_access_support_bi_weekly_digest` | Should only appear in digest |

## Test Events

All test events use the `ai` tag and have `Community` affiliation. They are created with different `field_choose_where_to_share_this` values to test visibility behavior.

| Title | Choose Where to Share | Date | Purpose |
|-------|----------------------|------|---------|
| Test Event - Main Page Only | `on_the_announcements_page` | +1 week | Should appear ONLY on `/events` |
| Test Event - Affinity Group Only | `on_your_affinity_group_page` | +2 weeks | Should appear ONLY on `/affinity-groups/access-support` |
| Test Event - Both Locations | Both options above | +3 weeks | Should appear on BOTH locations |
| Test Event - No Visibility | Empty array | +4 weeks | Should NOT appear on any listing pages |

## Using Fixtures in Cypress Tests

### Example: Testing Announcement Visibility

```javascript
describe('Announcement visibility based on choose_where_to_share_this', () => {
  it('Should show announcement on main page when configured', () => {
    cy.visit('/announcements');
    cy.contains('Test Announcement - Main Page Only').should('exist');
    cy.contains('Test Announcement - Affinity Group Only').should('not.exist');
  });

  it('Should show announcement on AG page when configured', () => {
    cy.visit('/affinity-groups/access-support');
    cy.contains('Test Announcement - Affinity Group Only').should('exist');
    cy.contains('Test Announcement - Main Page Only').should('not.exist');
  });

  it('Should show announcement on both locations when configured', () => {
    cy.visit('/announcements');
    cy.contains('Test Announcement - Both Locations').should('exist');

    cy.visit('/affinity-groups/access-support');
    cy.contains('Test Announcement - Both Locations').should('exist');
  });

  it('Should not show announcement with no visibility settings', () => {
    cy.visit('/announcements');
    cy.contains('Test Announcement - No Visibility').should('not.exist');

    cy.visit('/affinity-groups/access-support');
    cy.contains('Test Announcement - No Visibility').should('not.exist');
  });
});
```

### Example: Testing Event Visibility

```javascript
describe('Event visibility based on choose_where_to_share_this', () => {
  it('Should show event on main page when configured', () => {
    cy.visit('/events');
    cy.contains('Test Event - Main Page Only').should('exist');
    cy.contains('Test Event - Affinity Group Only').should('not.exist');
  });

  it('Should show event on AG page when configured', () => {
    cy.visit('/affinity-groups/access-support');
    cy.contains('Test Event - Affinity Group Only').should('exist');
    cy.contains('Test Event - Main Page Only').should('not.exist');
  });
});
```

## Complementary Approach: Form Tests

The fixtures are for **visibility/behavior testing**. You should still have separate tests that **create content via forms** to ensure forms work correctly:

```javascript
describe('Announcement form with choose_where_to_share_this field', () => {
  it('Should create announcement with field selections', () => {
    cy.loginAs('administrator@amptesting.com', 'password');
    cy.visit('/node/add/access_news');

    // Fill out form
    cy.get('#edit-title-0-value').type('Form Test Announcement');
    cy.get('#tag-ai').click();
    cy.get('#edit-field-affiliation').select('Community');

    // Test the choose_where_to_share_this field
    cy.get('input[value="on_the_announcements_page"]').should('be.checked'); // Default
    cy.get('input[value="on_your_affinity_group_page"]').check();

    cy.get('[name="moderation_state[0][state]"]').select('Published');
    cy.get('#edit-submit').click();

    cy.contains('Announcement Form Test Announcement has been created.');
  });
});
```

## Benefits

1. **Fast tests** - No need to create content in each test
2. **Consistent data** - Same fixtures every time
3. **Database refreshes** - Fixtures are recreated automatically
4. **Easy maintenance** - Update fixtures in one place
5. **Separate concerns** - Visibility tests use fixtures, form tests create content

## Updating Fixtures

To update the fixtures:

1. Edit the arrays in `amp_dev_install_create_test_announcements()` or `amp_dev_install_create_test_events()`
2. Reinstall the module: `drush pm:uninstall amp_dev && drush pm:install amp_dev`
3. Or run the database refresh process

## Notes

- Fixtures use the "ACCESS Support" affinity group (typically node ID 327)
- All announcements and events are published and owned by user 1
- Dates for events are relative (+1 week, +2 weeks, etc.) to ensure they're always in the future
- The fixtures check for existing content by title and delete it before recreating to avoid duplicates

## RP Account Panel Fixtures (D8-2719)

`walnut@pie.org` (uid 199) has pre-seeded data for the RP account sidebar panel:

- `field_xdusage_person_id = 99999` (fake; never matches real xdusage)
- Two rows in `access_user_rp_account` against Test Resource Alpha:
  - Grant `TST111`: balance 5,000 Core-hours, ends 2027-01-01
  - Grant `TST222`: balance 12,345.6789 Core-hours, ends 2026-12-31, long title for truncation testing
- The user-synced cache marker is set fresh so the page render path uses
  the seeded data rather than calling the live xdusage/identity APIs.

### Using the fixture in Cypress tests

```javascript
describe('RP account panel — populated', () => {
  beforeEach(() => {
    cy.loginAs('walnut@pie.org', 'Walnut');
    cy.visit('/documentation/resources/test-resource-alpha');
  });

  it('shows the populated panel', () => {
    cy.contains('YOUR ACCOUNT ON');
    cy.contains('walnut_alpha');
    cy.contains('TST111');
    cy.contains('Balance: 5,000 Core-hours');
  });
});
```

The fixture data is seeded by `amp_dev_install_create_rp_account_test_data()`
in `amp_dev.install`. To swap to fixture-based tests in this repo, the
DB artifact must be regenerated first to include the new fixtures.
