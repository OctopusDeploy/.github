# Trial Countdown Banner Visibility Fix

## Problem Analysis

The current implementation shows the "Trial Countdown" banner (displaying "Your cloud trial ends in X days") to all users, including those who:
- Are only in the "Everyone" team 
- Have no specific permissions assigned
- Cannot access the "License Usage" page that the banner links to

This creates a poor user experience where users see a banner with a link they cannot access.

## Solution Approach

The banner should only be displayed to users who have the necessary permissions to access the License Usage page. This ensures consistency between what users can see and what they can actually access.

## Implementation Steps

### 1. Identify Permission Requirements

First, locate where the License Usage page permissions are defined. Look for:

```typescript
// Example permission check for License Usage page
const canAccessLicenseUsage = user.hasPermission('License.View') || 
                             user.hasRole('InstanceManager') ||
                             user.hasRole('Administrator');
```

### 2. Update Trial Countdown Component

Modify the Trial Countdown banner component to include the same permission check:

```typescript
// Before (showing to all users)
const TrialCountdownBanner = () => {
  const { daysRemaining } = useTrialInfo();
  
  if (daysRemaining <= 0) return null;
  
  return (
    <div className="trial-countdown-banner">
      <span>Your cloud trial ends in {daysRemaining} days</span>
      <Link to="/license-usage">View License Usage</Link>
    </div>
  );
};

// After (showing only to users with proper permissions)
const TrialCountdownBanner = () => {
  const { daysRemaining } = useTrialInfo();
  const { user } = useCurrentUser();
  
  // Only show banner to users who can access License Usage page
  const canAccessLicenseUsage = user.hasPermission('License.View') || 
                               user.hasRole('InstanceManager') ||
                               user.hasRole('Administrator') ||
                               user.hasPermission('Instance.Manage');
  
  if (daysRemaining <= 0 || !canAccessLicenseUsage) return null;
  
  return (
    <div className="trial-countdown-banner">
      <span>Your cloud trial ends in {daysRemaining} days</span>
      <Link to="/license-usage">View License Usage</Link>
    </div>
  );
};
```

### 3. Alternative Approaches

#### Option A: Remove the Link for Unauthorized Users
If you want to show the countdown to all users but only show the link to authorized users:

```typescript
const TrialCountdownBanner = () => {
  const { daysRemaining } = useTrialInfo();
  const { user } = useCurrentUser();
  
  const canAccessLicenseUsage = user.hasPermission('License.View') || 
                               user.hasRole('InstanceManager') ||
                               user.hasRole('Administrator');
  
  if (daysRemaining <= 0) return null;
  
  return (
    <div className="trial-countdown-banner">
      <span>Your cloud trial ends in {daysRemaining} days</span>
      {canAccessLicenseUsage && (
        <Link to="/license-usage">View License Usage</Link>
      )}
    </div>
  );
};
```

#### Option B: Create a Shared Permission Hook
Create a reusable hook for consistent permission checking:

```typescript
// hooks/useLicensePermissions.ts
export const useLicensePermissions = () => {
  const { user } = useCurrentUser();
  
  return {
    canViewLicenseUsage: user.hasPermission('License.View') || 
                        user.hasRole('InstanceManager') ||
                        user.hasRole('Administrator') ||
                        user.hasPermission('Instance.Manage')
  };
};

// Component usage
const TrialCountdownBanner = () => {
  const { daysRemaining } = useTrialInfo();
  const { canViewLicenseUsage } = useLicensePermissions();
  
  if (daysRemaining <= 0 || !canViewLicenseUsage) return null;
  
  return (
    <div className="trial-countdown-banner">
      <span>Your cloud trial ends in {daysRemaining} days</span>
      <Link to="/license-usage">View License Usage</Link>
    </div>
  );
};
```

## Files to Locate and Modify

1. **Trial Countdown Component**: Look for files containing:
   - `TrialCountdown`
   - `trial-countdown`
   - "Your cloud trial ends"
   - "trial.*banner"

2. **License Usage Page**: Look for files containing:
   - `LicenseUsage`
   - `license-usage`
   - License permissions/routing

3. **Permission Definitions**: Look for:
   - Permission constants
   - Role definitions
   - Authorization utilities

## Search Commands

Use these commands to locate the relevant files:

```bash
# Find Trial Countdown component
grep -r "trial.*countdown\|Your cloud trial ends" --include="*.tsx" --include="*.ts" --include="*.jsx" --include="*.js"

# Find License Usage permissions
grep -r "License\.View\|LicenseUsage\|license-usage" --include="*.tsx" --include="*.ts" --include="*.jsx" --include="*.js"

# Find permission/authorization utilities
grep -r "hasPermission\|hasRole\|canAccess" --include="*.tsx" --include="*.ts" --include="*.jsx" --include="*.js"
```

## Testing

After implementing the fix:

1. **Test with Administrator/Manager**: Should see the banner with link
2. **Test with Regular User**: Should not see the banner at all
3. **Test with "Everyone" team only**: Should not see the banner
4. **Test License Usage page access**: Ensure users who can't see banner also can't access the page

## Expected Outcome

- Users without License Usage permissions will not see the Trial Countdown banner
- Users with proper permissions will continue to see the banner and can access the linked page
- Consistent user experience between what users can see and access
- No broken or inaccessible links for unauthorized users

## Next Steps

1. Locate the main Octopus Deploy application repository
2. Find the Trial Countdown banner component using the search commands above
3. Identify the License Usage page permission requirements
4. Implement the permission check in the banner component
5. Test with different user permission levels
6. Deploy and verify the fix in staging/production environments