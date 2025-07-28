# Racket Stringing App - Firebase Database & UI Documentation

## Overview
This is a comprehensive racket stringing service application built with Next.js, TypeScript, Firebase Firestore, and Tailwind CSS. The app provides real-time order tracking, customer management, and a modern user interface for both customers and service providers.

## Firebase Database Structure

### Collections

#### 1. Customers Collection
```typescript
interface Customer {
  id: string;
  name: string;           // Customer's full name
  email: string;          // Contact email
  phone: string;          // Phone number
  uid: string;            // Firebase Auth UID for security
}
```

#### 2. Orders Collection
```typescript
interface Order {
  id: string;
  customerId: string;     // Links to customer's uid
  racket: string;         // Racket model/brand
  string: string;         // String type/brand
  tension: string;        // String tension specification
  status: 'pending' | 'in_progress' | 'stringing' | 'quality_check' | 'completed' | 'ready_for_pickup';
  date: Timestamp;        // Order creation date
  estimatedDelivery: Timestamp; // Expected completion date
  notes?: string;         // Optional special instructions
}
```

#### 3. Payments Collection
```typescript
interface Payment {
  id: string;
  orderId: string;        // Links to order
  customerId: string;     // Links to customer
  amount: number;         // Payment amount
  paymentStatus: 'pending' | 'completed' | 'failed' | 'refunded';
  paymentDate: Timestamp; // Payment processing date
  paymentMethod: string;  // Payment method used
}
```

## Security Rules

The Firestore security rules ensure data privacy and security:

```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    // Customers: only authenticated user can access their document
    match /customers/{customerId} {
      allow read, write: if request.auth != null && request.auth.uid == customerId;
    }

    // Orders: only allow read/write if user owns the order
    match /orders/{orderId} {
      allow read, write: if request.auth != null && request.auth.uid == resource.data.customerId;
    }

    // Payments: customers can only read their own payment records
    match /payments/{paymentId} {
      allow read: if request.auth != null && request.auth.uid == resource.data.customerId;
      allow write: if false; // Server-side only for security
    }
  }
}
```

## JavaScript Functions

### Order Status Update Function
```typescript
export async function updateOrderStatus(orderId: string, newStatus: Order['status']) {
  try {
    const orderRef = doc(db, 'orders', orderId);
    await updateDoc(orderRef, { 
      status: newStatus,
      lastUpdated: Timestamp.now()
    });
    
    // Send notification to customer
    await sendNotification(orderId, newStatus);
    
    return { success: true, message: 'Order updated successfully' };
  } catch (error) {
    console.error("Error updating order:", error);
    return { success: false, message: 'Failed to update order' };
  }
}
```

### Notification System
The notification system supports multiple channels:
- Browser notifications (with user permission)
- Email notifications (placeholder for integration)
- SMS notifications (placeholder for integration)
- In-app alerts

## UX/UI Design - Order Status Screen Wireframe

### Screen Layout Description

#### Header Section
- **App Title**: "Racket Stringing Service" - Clean, bold typography
- **Subtitle**: "Track your order status in real-time" - Secondary text
- **Order ID**: Displayed prominently for reference

#### Progress Bar Section
```
┌─────────────────────────────────────────────────────────┐
│ Progress                                           85%   │
│ ████████████████████████████████████████████░░░░░░░░░   │
│                Quality Check                            │
└─────────────────────────────────────────────────────────┘
```
- **Visual Progress Bar**: Animated, color-coded progress indicator
  - Yellow: Pending/In Progress (0-40%)
  - Blue: Stringing/Processing (40-80%)
  - Green: Completed/Ready (80-100%)
- **Percentage Display**: Shows exact completion percentage
- **Status Text**: Current status in human-readable format

#### Order Details Card
```
┌─────────────────────────────────────────────────────────┐
│ Order Details                                           │
│ ─────────────────────────────────────────────────────── │
│ Racket:     Wilson Pro Staff 97                        │
│ String:     Luxilon ALU Power 125                      │
│ Tension:    55 lbs                                     │
│ Status:     Quality Check                              │
│ Notes:      Customer prefers cross strings at 53 lbs   │
└─────────────────────────────────────────────────────────┘
```
- **Clean Layout**: Two-column format with labels and values
- **Typography**: Clear hierarchy with bold labels
- **Background**: Subtle gray background for card separation

#### Estimated Delivery Section
```
┌─────────────────────────────────────────────────────────┐
│ Estimated Delivery                                      │
│ ─────────────────────────────────────────────────────── │
│ Friday, December 15, 2023 at 2:00 PM                  │
│ ✓ Ready for pickup now! (if completed)                 │
└─────────────────────────────────────────────────────────┘
```
- **Prominent Date Display**: Large, readable date format
- **Status Indicator**: Green checkmark when ready
- **Blue Background**: Distinguishes delivery information

#### Contact Support Button
```
┌─────────────────────────────────────────────────────────┐
│                   Contact Support                       │
└─────────────────────────────────────────────────────────┘
```
- **Full-Width Button**: Black background, white text
- **Hover Effects**: Smooth color transitions
- **Action**: Opens email client with pre-filled message

#### Additional Information Section
```
┌─────────────────────────────────────────────────────────┐
│ What happens next?                                      │
│ ─────────────────────────────────────────────────────── │
│   ①              ②              ③                      │
│ Inspection    Stringing    Quality Check               │
│ We inspect    Professional  Final inspection           │
│ your racket   stringing     and testing                │
└─────────────────────────────────────────────────────────┘
```
- **Process Steps**: Visual workflow with numbered circles
- **Color Coding**: Yellow, Blue, Green for different stages
- **Descriptive Text**: Brief explanation of each step

### Design Principles

#### Visual Hierarchy
1. **Primary**: Order status and progress (largest, most prominent)
2. **Secondary**: Order details and delivery date
3. **Tertiary**: Contact information and process steps

#### Color Scheme
- **Primary**: Black (#000000) for main actions and text
- **Secondary**: Gray shades for backgrounds and secondary text
- **Accent Colors**:
  - Yellow (#EAB308) for pending/early stages
  - Blue (#3B82F6) for active processing
  - Green (#10B981) for completion

#### Typography
- **Font Family**: Inter (Google Fonts) for clean, modern appearance
- **Font Weights**: 
  - Bold (700) for headings and important information
  - Medium (500) for labels and buttons
  - Regular (400) for body text

#### Spacing and Layout
- **Container**: Maximum width with centered alignment
- **Padding**: Consistent 1.5rem (24px) spacing
- **Card Spacing**: 2rem (32px) between major sections
- **Border Radius**: 0.5rem (8px) for modern, friendly appearance

#### Responsive Design
- **Mobile First**: Optimized for mobile devices
- **Breakpoints**: 
  - Mobile: Single column layout
  - Tablet: Maintains single column with larger spacing
  - Desktop: Enhanced spacing and typography

#### Interactive Elements
- **Hover States**: Subtle color transitions on buttons
- **Loading States**: Animated spinner for data fetching
- **Error States**: Clear error messages with recovery options
- **Real-time Updates**: Automatic refresh when order status changes

### Accessibility Features
- **Color Contrast**: WCAG AA compliant color combinations
- **Focus States**: Clear focus indicators for keyboard navigation
- **Screen Reader Support**: Semantic HTML and ARIA labels
- **Font Sizes**: Minimum 14px for readability

### User Experience Flow
1. **Entry**: Customer receives link with order ID
2. **Loading**: Smooth loading animation while fetching data
3. **Status Display**: Clear, immediate understanding of order progress
4. **Information Access**: Easy access to all relevant order details
5. **Support Contact**: One-click access to customer support
6. **Real-time Updates**: Automatic updates without page refresh

This wireframe design prioritizes clarity, accessibility, and user engagement while maintaining a professional, modern aesthetic suitable for a service-based business.

## Technical Implementation

### File Structure
```
src/
├── app/
│   ├── layout.tsx              # Root layout
│   ├── page.tsx                # Home page with demo
│   └── orders/
│       └── [orderId]/
│           └── page.tsx        # Order status page
├── components/
│   └── OrderStatus.tsx         # Order status component
└── lib/
    ├── firebase.ts             # Firebase configuration
    ├── orders.ts               # Order management functions
    └── notifications.ts        # Notification system
```

### Key Features
- **Real-time Updates**: Uses Firestore's `onSnapshot` for live data
- **Type Safety**: Full TypeScript implementation
- **Modern UI**: Tailwind CSS with responsive design
- **Security**: Firestore rules ensure data privacy
- **Notifications**: Multi-channel notification system
- **Error Handling**: Comprehensive error states and recovery

### Getting Started
1. Install dependencies: `npm install`
2. Configure Firebase project with provided credentials
3. Deploy Firestore security rules
4. Run development server: `npm run dev`
5. Create sample data using the demo interface
6. Test order status updates and notifications

This implementation provides a solid foundation for a production-ready racket stringing service application.
