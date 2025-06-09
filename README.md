Tuyệt vời! Là một lập trình viên React/TypeScript chuyên nghiệp, tôi sẽ tạo ra bộ code hoàn chỉnh và chất lượng cao cho Product Analytics Dashboard của bạn, tuân thủ mọi yêu cầu và các thực tiễn tốt nhất. Code sẽ được thiết kế để dễ đọc, dễ bảo trì và sẵn sàng để tích hợp vào dự án Vite/Tailwind/React Query hiện có của bạn.

Dưới đây là từng tệp code theo yêu cầu chi tiết.

---

### **1. Định nghĩa Data Models/Interfaces (TypeScript)**

**Tệp**: `src/types/productAnalytics.ts`

```typescript
// src/types/productAnalytics.ts

/**
 * Đại diện cho một điểm dữ liệu sử dụng sản phẩm hàng ngày.
 */
export interface ProductUsageDataPoint {
  date: string; // Định dạng: YYYY-MM-DD
  dailyActiveUsers: number;
  newUsers: number;
  churnedUsers: number;
}

/**
 * Đại diện cho dữ liệu sử dụng của một tính năng cụ thể.
 */
export interface FeatureUsageData {
  featureName: string;
  usageCount: number; // Tổng số lần sử dụng tính năng
  usersCount: number; // Số lượng người dùng đã sử dụng tính năng này
  averageSessionDuration: number; // Thời lượng phiên trung bình khi sử dụng tính năng này, tính bằng phút
}

/**
 * Đại diện cho dữ liệu tổng quan của Product Analytics.
 */
export interface ProductAnalyticsSummary {
  totalActiveUsers: number;
  totalFeaturesUsed: number;
  averageSessionDurationAcrossAllProducts: number; // Thời lượng phiên trung bình trên tất cả sản phẩm, tính bằng phút
}

/**
 * Kết hợp các kiểu dữ liệu trên thành một phản hồi API hoàn chỉnh cho Product Analytics.
 */
export type ProductAnalyticsResponse = {
  summary: ProductAnalyticsSummary;
  productUsageOverTime: ProductUsageDataPoint[];
  featureUsage: FeatureUsageData[];
};
```

### **2. Dữ liệu Mock (Mock Data)**

**Tệp**: `src/data/mockProductAnalyticsData.ts`

```typescript
// src/data/mockProductAnalyticsData.ts
import { format, subDays } from 'date-fns';
import { ProductAnalyticsResponse } from '../types/productAnalytics';

// Hàm helper để tạo dữ liệu sử dụng sản phẩm theo thời gian với xu hướng
const generateProductUsageData = (days: number): ProductAnalyticsResponse['productUsageOverTime'] => {
  const data: ProductAnalyticsResponse['productUsageOverTime'] = [];
  const today = new Date('2025-06-04'); // Ngày kết thúc cố định theo yêu cầu

  for (let i = days - 1; i >= 0; i--) {
    const date = subDays(today, i);
    const formattedDate = format(date, 'yyyy-MM-dd');

    // Tạo xu hướng tăng giảm nhẹ
    const baseActiveUsers = 500 + Math.floor(Math.sin(i / 5) * 100);
    const newUsers = 50 + Math.floor(Math.random() * 20);
    const churnedUsers = 20 + Math.floor(Math.random() * 10);

    data.push({
      date: formattedDate,
      dailyActiveUsers: Math.max(0, baseActiveUsers + newUsers - churnedUsers), // Đảm bảo không âm
      newUsers: newUsers,
      churnedUsers: churnedUsers,
    });
  }
  return data;
};

// Dữ liệu mock tuân thủ ProductAnalyticsResponse
export const mockProductAnalyticsData: ProductAnalyticsResponse = {
  summary: {
    totalActiveUsers: 152345,
    totalFeaturesUsed: 125,
    averageSessionDurationAcrossAllProducts: 45.7,
  },
  productUsageOverTime: generateProductUsageData(30), // Dữ liệu 30 ngày
  featureUsage: [
    {
      featureName: 'Product Search',
      usageCount: 891234,
      usersCount: 120567,
      averageSessionDuration: 3.2,
    },
    {
      featureName: 'Product Details Page',
      usageCount: 754321,
      usersCount: 98765,
      averageSessionDuration: 6.8,
    },
    {
      featureName: 'Add to Cart Button',
      usageCount: 234567,
      usersCount: 54321,
      averageSessionDuration: 1.5,
    },
    {
      featureName: 'Checkout Process',
      usageCount: 112345,
      usersCount: 32109,
      averageSessionDuration: 12.1,
    },
    {
      featureName: 'User Profile Settings',
      usageCount: 56789,
      usersCount: 21000,
      averageSessionDuration: 4.0,
    },
    {
      featureName: 'Chat with Seller',
      usageCount: 32109,
      usersCount: 15000,
      averageSessionDuration: 7.5,
    },
  ],
};
```

### **3. Dịch vụ API (Frontend)**

**Tệp**: `src/api/productAnalyticsService.ts`

```typescript
// src/api/productAnalyticsService.ts
import axios from 'axios';
import { ProductAnalyticsResponse } from '../types/productAnalytics';
import { mockProductAnalyticsData } from '../data/mockProductAnalyticsData';

// Định nghĩa BASE_URL cho API backend
// Sử dụng Vite environment variables
const BASE_URL = import.meta.env.VITE_API_BASE_URL || 'http://localhost:8000/api';

/**
 * Lấy dữ liệu phân tích sản phẩm từ API.
 * @param startDate Ngày bắt đầu (YYYY-MM-DD).
 * @param endDate Ngày kết thúc (YYYY-MM-DD).
 * @param productType Loại sản phẩm để lọc (tùy chọn).
 * @returns Promise giải quyết với dữ liệu ProductAnalyticsResponse.
 */
export const getProductAnalyticsData = async (
  startDate: string,
  endDate: string,
  productType?: string
): Promise<ProductAnalyticsResponse> => {
  // Mô phỏng độ trễ và trả về mock data trong môi trường phát triển
  if (import.meta.env.DEV) { // Sử dụng import.meta.env.DEV cho Vite dev environment
    console.log('Fetching mock product analytics data...');
    return new Promise((resolve) => {
      setTimeout(() => {
        resolve(mockProductAnalyticsData);
      }, 500); // Mô phỏng độ trễ 500ms
    });
  }

  // Môi trường production: gọi API thật
  try {
    const response = await axios.get<ProductAnalyticsResponse>(`${BASE_URL}/product-analytics/data`, {
      params: {
        startDate,
        endDate,
        productType: productType === 'All Products' ? undefined : productType, // Gửi undefined nếu là 'All Products'
      },
    });
    return response.data;
  } catch (error) {
    console.error('Error fetching product analytics data:', error);
    throw new Error('Failed to fetch product analytics data. Please try again.');
  }
};
```

### **4. Custom Hook (React Query)**

**Tệp**: `src/hooks/useProductAnalyticsData.ts`

```typescript
// src/hooks/useProductAnalyticsData.ts
import { useQuery } from '@tanstack/react-query';
import { getProductAnalyticsData } from '../api/productAnalyticsService';
import { ProductAnalyticsResponse } from '../types/productAnalytics';

/**
 * Custom hook để lấy dữ liệu phân tích sản phẩm bằng React Query.
 *
 * @param startDate Ngày bắt đầu của khoảng thời gian (YYYY-MM-DD).
 * @param endDate Ngày kết thúc của khoảng thời gian (YYYY-MM-DD).
 * @param productType Loại sản phẩm để lọc (tùy chọn, ví dụ: 'Product A', 'Product B', 'All Products').
 * @returns Đối tượng chứa dữ liệu, trạng thái tải, lỗi, v.v. từ useQuery.
 */
export const useProductAnalyticsData = (
  startDate: string,
  endDate: string,
  productType?: string
) => {
  const queryKey = ['productAnalytics', startDate, endDate, productType];

  return useQuery<ProductAnalyticsResponse, Error>({
    queryKey: queryKey,
    queryFn: () => getProductAnalyticsData(startDate, endDate, productType),
    staleTime: 5 * 60 * 1000, // Dữ liệu được coi là stale sau 5 phút (5 * 60 * 1000 ms)
    gcTime: 10 * 60 * 1000,   // Dữ liệu sẽ được dọn dẹp khỏi cache sau 10 phút (10 * 60 * 1000 ms) nếu không có query nào sử dụng
    refetchOnWindowFocus: false, // Không tự động refetch khi tab được focus lại
    refetchOnReconnect: false,   // Không tự động refetch khi kết nối mạng được khôi phục
  });
};
```

### **5. Thành phần UI (Components)**

#### a. Thành phần `DateRangePicker`

**Tệp**: `src/components/common/DateRangePicker.tsx`

```typescript
// src/components/common/DateRangePicker.tsx
import React from 'react';

/**
 * Props cho DateRangePicker component.
 */
interface DateRangePickerProps {
  onDateRangeChange: (start: string, end: string) => void;
  startDate: string;
  endDate: string;
}

/**
 * Component cho phép chọn ngày bắt đầu và ngày kết thúc cho một khoảng thời gian.
 * Sử dụng input type="date" đơn giản.
 */
const DateRangePicker: React.FC<DateRangePickerProps> = ({
  onDateRangeChange,
  startDate,
  endDate,
}) => {
  const handleStartDateChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    onDateRangeChange(e.target.value, endDate);
  };

  const handleEndDateChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    onDateRangeChange(startDate, e.target.value);
  };

  return (
    <div className="flex flex-col sm:flex-row items-center space-y-2 sm:space-y-0 sm:space-x-4">
      <div className="flex items-center space-x-2">
        <label htmlFor="startDate" className="text-gray-700">
          From:
        </label>
        <input
          type="date"
          id="startDate"
          value={startDate}
          onChange={handleStartDateChange}
          className="p-2 border border-gray-300 rounded-md focus:ring-blue-500 focus:border-blue-500"
        />
      </div>
      <div className="flex items-center space-x-2">
        <label htmlFor="endDate" className="text-gray-700">
          To:
        </label>
        <input
          type="date"
          id="endDate"
          value={endDate}
          onChange={handleEndDateChange}
          className="p-2 border border-gray-300 rounded-md focus:ring-blue-500 focus:border-blue-500"
        />
      </div>
    </div>
  );
};

export default DateRangePicker;
```

#### b. Thành phần `Spinner`

**Tệp**: `src/components/common/Spinner.tsx`

```typescript
// src/components/common/Spinner.tsx
import React from 'react';

/**
 * Props cho Spinner component.
 */
interface SpinnerProps {
  className?: string; // Tailwind CSS classes tùy chỉnh cho spinner container.
}

/**
 * Component hiển thị một animation spinner CSS đơn giản.
 */
const Spinner: React.FC<SpinnerProps> = ({ className = '' }) => {
  return (
    <div
      className={`
        inline-block
        w-8 h-8
        border-4 border-t-4 border-gray-200 border-t-blue-500
        rounded-full animate-spin
        ${className}
      `}
      role="status"
      aria-label="Loading..."
    >
      <span className="sr-only">Loading...</span> {/* Screen reader text */}
    </div>
  );
};

export default Spinner;
```

#### c. Thành phần `ErrorMessage`

**Tệp**: `src/components/common/ErrorMessage.tsx`

```typescript
// src/components/common/ErrorMessage.tsx
import React from 'react';

/**
 * Props cho ErrorMessage component.
 */
interface ErrorMessageProps {
  message: string;
}

/**
 * Component hiển thị thông báo lỗi thân thiện.
 */
const ErrorMessage: React.FC<ErrorMessageProps> = ({ message }) => {
  return (
    <div className="p-4 mb-4 text-sm text-red-700 bg-red-100 rounded-lg" role="alert">
      <p className="font-medium">Error:</p>
      <p>{message}</p>
    </div>
  );
};

export default ErrorMessage;
```

#### d. Thành phần `ProductTypeFilter`

**Tệp**: `src/components/productAnalytics/ProductTypeFilter.tsx`

```typescript
// src/components/productAnalytics/ProductTypeFilter.tsx
import React from 'react';

/**
 * Props cho ProductTypeFilter component.
 */
interface ProductTypeFilterProps {
  onSelect: (productType: string) => void;
  selectedType: string;
}

/**
 * Component cung cấp lựa chọn loại sản phẩm để lọc dữ liệu.
 */
const ProductTypeFilter: React.FC<ProductTypeFilterProps> = ({
  onSelect,
  selectedType,
}) => {
  const productTypes = ['All Products', 'Product A', 'Product B', 'Product C', 'Service X']; // Các lựa chọn giả định

  const handleChange = (e: React.ChangeEvent<HTMLSelectElement>) => {
    onSelect(e.target.value);
  };

  return (
    <div>
      <label htmlFor="productType" className="block text-sm font-medium text-gray-700 mb-1">
        Product Type:
      </label>
      <select
        id="productType"
        value={selectedType}
        onChange={handleChange}
        className="block w-full p-2 border border-gray-300 rounded-md shadow-sm focus:ring-blue-500 focus:border-blue-500 sm:text-sm"
      >
        {productTypes.map((type) => (
          <option key={type} value={type}>
            {type}
          </option>
        ))}
      </select>
    </div>
  );
};

export default ProductTypeFilter;
```

#### e. Thành phần `DailyActiveUsersChart`

**Tệp**: `src/components/productAnalytics/DailyActiveUsersChart.tsx`

```typescript
// src/components/productAnalytics/DailyActiveUsersChart.tsx
import React from 'react';
import {
  LineChart,
  Line,
  XAxis,
  YAxis,
  CartesianGrid,
  Tooltip,
  Legend,
  ResponsiveContainer,
} from 'recharts';
import { format } from 'date-fns';
import { ProductUsageDataPoint } from '../../types/productAnalytics';

/**
 * Props cho DailyActiveUsersChart component.
 */
interface DailyActiveUsersChartProps {
  data: ProductUsageDataPoint[];
}

/**
 * Component hiển thị biểu đồ đường cho dữ liệu sử dụng sản phẩm hàng ngày.
 */
const DailyActiveUsersChart: React.FC<DailyActiveUsersChartProps> = ({ data }) => {
  // Định dạng ngày tháng cho trục X
  const dateFormatter = (isoDate: string) => {
    return format(new Date(isoDate), 'MMM dd');
  };

  return (
    <div className="bg-white p-6 rounded-lg shadow-md mb-6">
      <h3 className="text-xl font-semibold text-gray-800 mb-4">Daily Product Usage</h3>
      <ResponsiveContainer width="100%" height={300}>
        <LineChart
          data={data}
          margin={{
            top: 5,
            right: 30,
            left: 20,
            bottom: 5,
          }}
        >
          <CartesianGrid strokeDasharray="3 3" stroke="#e0e0e0" />
          <XAxis dataKey="date" tickFormatter={dateFormatter} />
          <YAxis />
          <Tooltip />
          <Legend />
          <Line
            type="monotone"
            dataKey="dailyActiveUsers"
            stroke="#8884d8"
            name="Daily Active Users"
            strokeWidth={2}
          />
          <Line
            type="monotone"
            dataKey="newUsers"
            stroke="#82ca9d"
            name="New Users"
            strokeWidth={2}
          />
          <Line
            type="monotone"
            dataKey="churnedUsers"
            stroke="#ff7300"
            name="Churned Users"
            strokeWidth={2}
          />
        </LineChart>
      </ResponsiveContainer>
    </div>
  );
};

export default DailyActiveUsersChart;
```

#### f. Thành phần `FeatureUsageTable`

**Tệp**: `src/components/productAnalytics/FeatureUsageTable.tsx`

```typescript
// src/components/productAnalytics/FeatureUsageTable.tsx
import React from 'react';
import { FeatureUsageData } from '../../types/productAnalytics';

/**
 * Props cho FeatureUsageTable component.
 */
interface FeatureUsageTableProps {
  data: FeatureUsageData[];
}

/**
 * Component hiển thị một bảng sử dụng tính năng.
 */
const FeatureUsageTable: React.FC<FeatureUsageTableProps> = ({ data }) => {
  return (
    <div className="bg-white p-6 rounded-lg shadow-md mb-6">
      <h3 className="text-xl font-semibold text-gray-800 mb-4">Feature Usage Breakdown</h3>
      <div className="overflow-x-auto"> {/* Để xử lý bảng trên màn hình nhỏ */}
        <table className="table-auto w-full text-left border-collapse">
          <thead>
            <tr className="bg-gray-100 text-gray-700 uppercase text-sm leading-normal">
              <th className="py-3 px-6 border-b border-gray-200">Feature Name</th>
              <th className="py-3 px-6 border-b border-gray-200">Usage Count</th>
              <th className="py-3 px-6 border-b border-gray-200">Users Count</th>
              <th className="py-3 px-6 border-b border-gray-200">Avg Session Duration (min)</th>
            </tr>
          </thead>
          <tbody className="text-gray-600 text-sm font-light">
            {data.map((feature) => (
              <tr key={feature.featureName} className="border-b border-gray-200 hover:bg-gray-50">
                <td className="py-3 px-6 whitespace-nowrap">{feature.featureName}</td>
                <td className="py-3 px-6">{feature.usageCount.toLocaleString()}</td>
                <td className="py-3 px-6">{feature.usersCount.toLocaleString()}</td>
                <td className="py-3 px-6">{feature.averageSessionDuration.toFixed(1)}</td>
              </tr>
            ))}
          </tbody>
        </table>
      </div>
    </div>
  );
};

export default FeatureUsageTable;
```

#### g. Thành phần `ProductAnalyticsDashboard`

**Tệp**: `src/components/productAnalytics/ProductAnalyticsDashboard.tsx`

```typescript
// src/components/productAnalytics/ProductAnalyticsDashboard.tsx
import React, { useState } from 'react';
import { format, subDays } from 'date-fns';
import { useProductAnalyticsData } from '../../hooks/useProductAnalyticsData';
import Spinner from '../common/Spinner';
import ErrorMessage from '../common/ErrorMessage';
import DateRangePicker from '../common/DateRangePicker';
import DailyActiveUsersChart from './DailyActiveUsersChart';
import FeatureUsageTable from './FeatureUsageTable';
import ProductTypeFilter from './ProductTypeFilter';

// Component Card đơn giản
const Card: React.FC<{ title: string; value: string | number; className?: string }> = ({
  title,
  value,
  className = '',
}) => (
  <div className={`bg-white p-6 rounded-lg shadow-md ${className}`}>
    <h4 className="text-lg font-semibold text-gray-600 mb-2">{title}</h4>
    <p className="text-3xl font-bold text-gray-800">{value}</p>
  </div>
);

/**
 * Component chính cho Product Analytics Dashboard.
 * Hiển thị dữ liệu tổng quan, biểu đồ và bảng sử dụng tính năng.
 */
const ProductAnalyticsDashboard: React.FC = () => {
  // Khởi tạo ngày bắt đầu và kết thúc (khoảng 30 ngày trước đến ngày hiện tại)
  const defaultEndDate = format(new Date('2025-06-04'), 'yyyy-MM-dd'); // Cố định ngày theo mock data
  const defaultStartDate = format(subDays(new Date('2025-06-04'), 29), 'yyyy-MM-dd'); // 30 ngày trước đó

  const [startDate, setStartDate] = useState<string>(defaultStartDate);
  const [endDate, setEndDate] = useState<string>(defaultEndDate);
  const [selectedProductType, setSelectedProductType] = useState<string>('All Products');

  const { data, isLoading, isError, error, isFetching } = useProductAnalyticsData(
    startDate,
    endDate,
    selectedProductType
  );

  const handleDateRangeChange = (start: string, end: string) => {
    // Đảm bảo startDate không lớn hơn endDate
    if (new Date(start) > new Date(end)) {
        alert('Start date cannot be after end date.');
        return;
    }
    setStartDate(start);
    setEndDate(end);
  };

  if (isError) {
    return (
      <div className="min-h-screen flex items-center justify-center p-4 bg-gray-100">
        <ErrorMessage message={error?.message || 'Something went wrong while fetching data.'} />
      </div>
    );
  }

  return (
    <div className="min-h-screen bg-gray-100 p-4 sm:p-6 lg:p-8">
      <h1 className="text-3xl font-bold text-gray-800 mb-6">Product Analytics Dashboard</h1>

      {/* Control Panel: Date Range and Product Type Filter */}
      <div className="bg-white p-6 rounded-lg shadow-md mb-6 flex flex-col sm:flex-row items-start sm:items-center justify-between space-y-4 sm:space-y-0">
        <DateRangePicker
          startDate={startDate}
          endDate={endDate}
          onDateRangeChange={handleDateRangeChange}
        />
        <ProductTypeFilter
          selectedType={selectedProductType}
          onSelect={setSelectedProductType}
        />
      </div>

      {isLoading || isFetching ? (
        <div className="flex justify-center items-center h-64">
          <Spinner className="w-12 h-12" />
          <p className="ml-4 text-lg text-gray-600">Loading analytics data...</p>
        </div>
      ) : (
        <>
          {/* Summary Cards */}
          <div className="grid grid-cols-1 md:grid-cols-3 gap-4 mb-6">
            <Card
              title="Total Active Users (Overall)"
              value={data?.summary.totalActiveUsers.toLocaleString() || 'N/A'}
            />
            <Card
              title="Total Features Used (Overall)"
              value={data?.summary.totalFeaturesUsed.toLocaleString() || 'N/A'}
            />
            <Card
              title="Avg Session Duration (All Products)"
              value={`${data?.summary.averageSessionDurationAcrossAllProducts.toFixed(1)} min` || 'N/A'}
            />
          </div>

          {/* Daily Active Users Chart */}
          {data?.productUsageOverTime && data.productUsageOverTime.length > 0 ? (
            <DailyActiveUsersChart data={data.productUsageOverTime} />
          ) : (
            <div className="bg-white p-6 rounded-lg shadow-md mb-6 text-center text-gray-500">
              No daily usage data available for the selected period.
            </div>
          )}

          {/* Feature Usage Table */}
          {data?.featureUsage && data.featureUsage.length > 0 ? (
            <FeatureUsageTable data={data.featureUsage} />
          ) : (
            <div className="bg-white p-6 rounded-lg shadow-md mb-6 text-center text-gray-500">
              No feature usage data available for the selected period.
            </div>
          )}
        </>
      )}
    </div>
  );
};

export default ProductAnalyticsDashboard;
```

### **6. Cập nhật `App.tsx`**

**Tệp**: `src/App.tsx`

```typescript
// src/App.tsx
import React from 'react';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import ProductAnalyticsDashboard from './components/productAnalytics/ProductAnalyticsDashboard';

// Tạo một instance của QueryClient
const queryClient = new QueryClient();

const App: React.FC = () => {
  return (
    <QueryClientProvider client={queryClient}>
      <div className="App">
        <ProductAnalyticsDashboard />
      </div>
    </QueryClientProvider>
  );
};

export default App;
```

### **7. Cập nhật `main.tsx`**

**Tệp**: `src/main.tsx`

```typescript
// src/main.tsx
import React from 'react';
import ReactDOM from 'react-dom/client';
import App from './App';
import './index.css'; // Import Tailwind CSS và global styles

ReactDOM.createRoot(document.getElementById('root')!).render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
);
```

### **8. Cấu hình Vite Environment Variable**

Nếu bạn muốn test môi trường production hoặc đổi API BASE URL, bạn cần cấu hình `VITE_API_BASE_URL` trong file `.env` ở thư mục gốc của dự án.

**Tệp**: `.env`

```
VITE_API_BASE_URL=http://your-production-api.com/api
```

---

### **Hướng dẫn chạy dự án**

1.  **Đảm bảo môi trường:**
    *   Bạn đã cài đặt Node.js và npm/yarn.
    *   Bạn đã có một dự án Vite cơ bản với React, TypeScript, Tailwind CSS, Axios, React Query đã được cấu hình.

2.  **Cài đặt các thư viện bổ sung:**
    Nếu chưa có, hãy cài đặt `recharts` và `date-fns`:
    ```bash
    npm install recharts date-fns
    # hoặc
    yarn add recharts date-fns
    ```

3.  **Sao chép và dán code:**
    Dán các tệp code đã tạo vào đúng cấu trúc thư mục của dự án Vite của bạn như đã giả định ở trên (`src/api`, `src/components`, `src/hooks`, `src/data`, `src/types`, `App.tsx`, `main.tsx`).

4.  **Chạy ứng dụng:**
    ```bash
    npm run dev
    # hoặc
    yarn dev
    ```
