# Admin Dashboard Documentation

## Overview

The admin dashboard is an integrated part of the main web application, providing administrative users with tools to manage content, users, and system activities. It follows a single-page application (SPA) architecture within the main React frontend.

## Architecture Decision

### Integrated Approach (Chosen)
The admin dashboard is implemented as protected routes within the main application rather than as a separate application. This decision provides:

**Advantages:**
- Single codebase to maintain
- Shared components and design system
- Unified authentication flow
- Simplified deployment process
- Consistent user experience
- Shared state management

**Trade-offs:**
- Larger bundle size (mitigated by code splitting)
- Admin code shipped to all users (secured by route guards)

## Access Control

### Route Protection
```typescript
// src/guards/AdminRoute.tsx
const AdminRoute = ({ children }: { children: React.ReactNode }) => {
  const { user, isLoaded } = useUser();
  
  if (!isLoaded) return <LoadingScreen />;
  
  if (!user || user.publicMetadata.role !== 'ADMIN') {
    return <Navigate to="/" replace />;
  }
  
  return <>{children}</>;
};
```

### URL Structure
- Base URL: `/admin`
- Dashboard: `/admin/dashboard`
- Items Management: `/admin/items`
- Users Management: `/admin/users`
- Activities: `/admin/activities`
- Settings: `/admin/settings`

## Component Structure

### Layout Components
```
src/components/admin/
├── AdminLayout.tsx        # Main admin layout wrapper
├── AdminSidebar.tsx       # Navigation sidebar
├── AdminHeader.tsx        # Top navigation bar
├── AdminBreadcrumb.tsx    # Breadcrumb navigation
└── AdminFooter.tsx        # Footer with version info
```

### Feature Components
```
src/components/admin/
├── dashboard/
│   ├── StatCard.tsx       # Statistics display card
│   ├── RecentActivity.tsx # Recent activities list
│   ├── QuickActions.tsx   # Quick action buttons
│   └── Analytics.tsx      # Charts and graphs
├── data-table/
│   ├── DataTable.tsx      # Reusable data table
│   ├── TablePagination.tsx
│   ├── TableFilters.tsx
│   └── BulkActions.tsx
└── forms/
    ├── ItemForm.tsx       # Item creation/editing
    ├── UserForm.tsx       # User management form
    └── SettingsForm.tsx   # System settings
```

## Page Structure

### 1. Dashboard Page (`/admin/dashboard`)
**Purpose**: Overview of system status and quick access to common tasks

**Features:**
- Key metrics cards (total items, users, pending activities)
- Recent activity feed
- Quick action buttons
- Performance charts
- System health indicators

### 2. Items Management (`/admin/items`)
**Purpose**: CRUD operations for content items

**Features:**
- Searchable/sortable data table
- Create new item button
- Bulk operations (delete, export)
- Category filtering
- Image upload with preview
- Rich text editor for descriptions
- Inventory/availability management

### 3. Users Management (`/admin/users`)
**Purpose**: User administration and role management

**Features:**
- User listing with search
- Role assignment (USER/ADMIN)
- User activity history
- Account status management
- Export user data
- Send notifications

### 4. Activities Management (`/admin/activities`)
**Purpose**: Monitor and manage user activities

**Features:**
- Activity listing with filters
- Status updates (PENDING → PROCESSING → COMPLETED)
- Activity details modal
- Export functionality
- Bulk status updates
- Activity timeline view

### 5. Settings (`/admin/settings`)
**Purpose**: System configuration and preferences

**Features:**
- General settings
- Email templates
- System preferences
- API configuration
- Backup management
- Audit logs

## API Integration

### Admin-Specific Endpoints
```typescript
// src/services/admin.ts
const adminApi = {
  // Dashboard
  getDashboardStats: () => api.get('/api/admin/dashboard'),
  
  // Items Management
  createItem: (data) => api.post('/api/items', data),
  updateItem: (id, data) => api.put(`/api/items/${id}`, data),
  deleteItem: (id) => api.delete(`/api/items/${id}`),
  bulkDeleteItems: (ids) => api.post('/api/items/bulk-delete', { ids }),
  
  // Users Management
  getUsers: (params) => api.get('/api/admin/users', { params }),
  updateUserRole: (id, role) => api.put(`/api/admin/users/${id}/role`, { role }),
  
  // Activities
  getActivities: (params) => api.get('/api/admin/activities', { params }),
  updateActivityStatus: (id, status) => api.put(`/api/activities/${id}/status`, { status }),
  
  // Analytics
  getAnalytics: (params) => api.get('/api/admin/analytics', { params }),
};
```

## State Management

### Admin Store (Zustand)
```typescript
// src/store/adminStore.ts
interface AdminStore {
  // Dashboard
  stats: DashboardStats | null;
  recentActivities: Activity[];
  
  // UI State
  sidebarCollapsed: boolean;
  selectedItems: string[];
  
  // Actions
  fetchDashboardStats: () => Promise<void>;
  toggleSidebar: () => void;
  setSelectedItems: (items: string[]) => void;
}
```

## Common Patterns

### Data Table Usage
```typescript
<DataTable
  data={items}
  columns={itemColumns}
  searchable
  sortable
  selectable
  onSelectionChange={setSelectedItems}
  actions={
    <BulkActions
      selectedCount={selectedItems.length}
      onDelete={handleBulkDelete}
      onExport={handleExport}
    />
  }
/>
```

### Form Handling
```typescript
const ItemForm = () => {
  const { register, handleSubmit, formState: { errors } } = useForm<ItemFormData>({
    resolver: zodResolver(itemSchema),
  });
  
  const onSubmit = async (data: ItemFormData) => {
    try {
      await adminApi.createItem(data);
      toast.success('Item created successfully');
      navigate('/admin/items');
    } catch (error) {
      toast.error('Failed to create item');
    }
  };
  
  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      {/* Form fields */}
    </form>
  );
};
```

## Security Considerations

### Frontend Security
1. **Route Guards**: All admin routes protected by authentication check
2. **Role Verification**: User role checked before rendering admin components
3. **API Token**: Admin requests include authentication token
4. **Input Validation**: Client-side validation for all forms
5. **XSS Prevention**: React's built-in protection + sanitization

### Backend Security
1. **Middleware Protection**: Admin endpoints require admin role
2. **Rate Limiting**: Stricter limits for admin endpoints
3. **Audit Logging**: All admin actions logged
4. **Input Validation**: Server-side validation with Pydantic
5. **CORS Configuration**: Restricted origins

## UI/UX Guidelines

### Design Principles
1. **Consistency**: Use the same design system as the main app
2. **Clarity**: Clear labels and intuitive navigation
3. **Efficiency**: Quick access to common tasks
4. **Feedback**: Loading states and success/error messages
5. **Responsive**: Works on tablet and desktop devices

### Color Scheme
- Primary actions: Brand primary color
- Danger actions: Red (#EF4444)
- Success states: Green (#10B981)
- Warning states: Yellow (#F59E0B)
- Neutral: Gray scale from design system

### Typography
- Headings: Inter font family
- Body text: System font stack
- Monospace: For IDs and technical data

## Performance Optimization

### Code Splitting
```typescript
// Lazy load admin routes
const AdminDashboard = lazy(() => import('./pages/admin/Dashboard'));
const AdminItems = lazy(() => import('./pages/admin/ItemsManagement'));
const AdminUsers = lazy(() => import('./pages/admin/UsersManagement'));
```

### Data Caching
- Use TanStack Query for API response caching
- Implement pagination for large datasets
- Virtual scrolling for long lists
- Debounced search inputs

### Image Optimization
- Lazy load images in data tables
- Thumbnail generation for previews
- WebP format with fallbacks
- CDN delivery for production

## Development Workflow

### Adding New Admin Features
1. Create page component in `src/pages/admin/`
2. Add route to admin router
3. Create necessary API endpoints
4. Add admin middleware to backend
5. Update admin navigation
6. Add feature to admin dashboard

### Testing Admin Features
1. Unit tests for components
2. Integration tests for API endpoints
3. E2E tests for critical workflows
4. Role-based access testing
5. Performance testing for data tables

## Common Tasks

### Adding a New Admin Page
```typescript
// 1. Create the page component
const NewFeature = () => {
  return (
    <AdminLayout>
      <PageHeader title="New Feature" />
      {/* Page content */}
    </AdminLayout>
  );
};

// 2. Add to router
<Route path="/admin/new-feature" element={
  <AdminRoute>
    <NewFeature />
  </AdminRoute>
} />

// 3. Add to sidebar navigation
const adminNavItems = [
  // ... existing items
  {
    label: 'New Feature',
    icon: <IconName />,
    path: '/admin/new-feature',
  },
];
```

### Creating Admin API Endpoint
```python
# backend/app/api/v1/admin/new_feature.py
from fastapi import APIRouter, Depends
from app.core.deps import get_current_admin_user

router = APIRouter()

@router.get("/")
async def get_new_feature_data(
    current_user = Depends(get_current_admin_user)
):
    # Admin-only logic here
    return {"data": "admin only data"}
```

## Troubleshooting

### Common Issues
1. **Admin route not accessible**: Check user role in Clerk dashboard
2. **API returns 403**: Verify JWT includes admin role claim
3. **Slow data tables**: Implement pagination and virtual scrolling
4. **Bundle size concerns**: Ensure admin routes are lazy loaded

### Debug Mode
Enable admin debug mode in development:
```typescript
// .env.local
VITE_ADMIN_DEBUG=true
```

This will show:
- User role information
- API request/response logs
- Performance metrics
- Component render counts