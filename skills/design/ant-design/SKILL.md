---
name: ant-design
description: Builds enterprise React applications with Ant Design's comprehensive component library. Use when creating admin dashboards, data tables, complex forms, or enterprise UIs with consistent design language.
---

# Ant Design

Enterprise-class React UI library with comprehensive components for admin interfaces.

## Quick Start

```bash
npm install antd
```

```tsx
import React from 'react';
import { ConfigProvider, Button, Space } from 'antd';

function App() {
  return (
    <ConfigProvider
      theme={{
        token: {
          colorPrimary: '#1677ff',
        },
      }}
    >
      <Space>
        <Button type="primary">Primary</Button>
        <Button>Default</Button>
      </Space>
    </ConfigProvider>
  );
}
```

## Core Components

### Button

```tsx
import { Button, Space } from 'antd';
import { SearchOutlined, DownloadOutlined } from '@ant-design/icons';

// Types
<Button type="primary">Primary</Button>
<Button type="default">Default</Button>
<Button type="dashed">Dashed</Button>
<Button type="text">Text</Button>
<Button type="link">Link</Button>

// Sizes
<Button size="large">Large</Button>
<Button size="middle">Middle</Button>
<Button size="small">Small</Button>

// States
<Button loading>Loading</Button>
<Button disabled>Disabled</Button>
<Button danger>Danger</Button>

// With icons
<Button type="primary" icon={<SearchOutlined />}>Search</Button>
<Button icon={<DownloadOutlined />} />
```

### Form

```tsx
import { Form, Input, Select, Button, Checkbox, DatePicker } from 'antd';

function MyForm() {
  const [form] = Form.useForm();

  const onFinish = (values) => {
    console.log('Form values:', values);
  };

  return (
    <Form
      form={form}
      layout="vertical"
      onFinish={onFinish}
      initialValues={{ remember: true }}
    >
      <Form.Item
        label="Email"
        name="email"
        rules={[
          { required: true, message: 'Please input your email!' },
          { type: 'email', message: 'Invalid email format' },
        ]}
      >
        <Input placeholder="you@example.com" />
      </Form.Item>

      <Form.Item
        label="Password"
        name="password"
        rules={[{ required: true, min: 8 }]}
      >
        <Input.Password />
      </Form.Item>

      <Form.Item label="Role" name="role">
        <Select>
          <Select.Option value="admin">Admin</Select.Option>
          <Select.Option value="user">User</Select.Option>
        </Select>
      </Form.Item>

      <Form.Item label="Date" name="date">
        <DatePicker style={{ width: '100%' }} />
      </Form.Item>

      <Form.Item name="remember" valuePropName="checked">
        <Checkbox>Remember me</Checkbox>
      </Form.Item>

      <Form.Item>
        <Button type="primary" htmlType="submit">Submit</Button>
      </Form.Item>
    </Form>
  );
}
```

### Table

```tsx
import { Table, Tag, Space, Button } from 'antd';

const columns = [
  {
    title: 'Name',
    dataIndex: 'name',
    key: 'name',
    sorter: (a, b) => a.name.localeCompare(b.name),
    render: (text) => <a>{text}</a>,
  },
  {
    title: 'Age',
    dataIndex: 'age',
    key: 'age',
    sorter: (a, b) => a.age - b.age,
  },
  {
    title: 'Status',
    dataIndex: 'status',
    key: 'status',
    filters: [
      { text: 'Active', value: 'active' },
      { text: 'Inactive', value: 'inactive' },
    ],
    onFilter: (value, record) => record.status === value,
    render: (status) => (
      <Tag color={status === 'active' ? 'green' : 'red'}>
        {status.toUpperCase()}
      </Tag>
    ),
  },
  {
    title: 'Action',
    key: 'action',
    render: (_, record) => (
      <Space size="middle">
        <a>Edit</a>
        <a>Delete</a>
      </Space>
    ),
  },
];

<Table
  columns={columns}
  dataSource={data}
  rowKey="id"
  pagination={{ pageSize: 10 }}
  rowSelection={{
    type: 'checkbox',
    onChange: (selectedRowKeys) => console.log(selectedRowKeys),
  }}
/>
```

### Modal

```tsx
import { Modal, Button } from 'antd';
import { useState } from 'react';
import { ExclamationCircleFilled } from '@ant-design/icons';

function ModalDemo() {
  const [open, setOpen] = useState(false);

  return (
    <>
      <Button onClick={() => setOpen(true)}>Open Modal</Button>
      <Modal
        title="Modal Title"
        open={open}
        onOk={() => setOpen(false)}
        onCancel={() => setOpen(false)}
        okText="Confirm"
        cancelText="Cancel"
      >
        <p>Modal content here...</p>
      </Modal>
    </>
  );
}

// Confirmation modal
const { confirm } = Modal;

function showConfirm() {
  confirm({
    title: 'Do you want to delete these items?',
    icon: <ExclamationCircleFilled />,
    content: 'This action cannot be undone.',
    onOk() {
      return deleteItems();
    },
    onCancel() {},
  });
}
```

### Notification & Message

```tsx
import { notification, message, Button, Space } from 'antd';

// Notification (corner popup)
const [api, contextHolder] = notification.useNotification();

const openNotification = () => {
  api.success({
    message: 'Success',
    description: 'Your changes have been saved.',
    placement: 'topRight',
    duration: 4.5,
  });
};

<>
  {contextHolder}
  <Button onClick={openNotification}>Show Notification</Button>
</>

// Message (top center toast)
const [messageApi, messageContextHolder] = message.useMessage();

const showMessage = () => {
  messageApi.success('Operation completed successfully');
  messageApi.error('Something went wrong');
  messageApi.warning('Please check your input');
  messageApi.loading('Loading...');
};
```

### Menu and Navigation

```tsx
import { Menu, Breadcrumb } from 'antd';
import {
  HomeOutlined,
  SettingOutlined,
  UserOutlined,
} from '@ant-design/icons';

// Sidebar menu
const menuItems = [
  {
    key: 'home',
    icon: <HomeOutlined />,
    label: 'Home',
  },
  {
    key: 'users',
    icon: <UserOutlined />,
    label: 'Users',
    children: [
      { key: 'list', label: 'User List' },
      { key: 'add', label: 'Add User' },
    ],
  },
  {
    key: 'settings',
    icon: <SettingOutlined />,
    label: 'Settings',
  },
];

<Menu
  mode="inline"
  defaultSelectedKeys={['home']}
  defaultOpenKeys={['users']}
  items={menuItems}
  onClick={({ key }) => navigate(key)}
/>

// Breadcrumb
<Breadcrumb
  items={[
    { href: '/', title: <HomeOutlined /> },
    { href: '/users', title: 'Users' },
    { title: 'John Doe' },
  ]}
/>
```

### Tabs

```tsx
import { Tabs } from 'antd';

<Tabs
  defaultActiveKey="1"
  items={[
    {
      key: '1',
      label: 'Tab 1',
      children: 'Content of Tab 1',
    },
    {
      key: '2',
      label: 'Tab 2',
      children: 'Content of Tab 2',
    },
    {
      key: '3',
      label: 'Tab 3',
      children: 'Content of Tab 3',
      disabled: true,
    },
  ]}
  onChange={(key) => console.log(key)}
/>
```

### Card

```tsx
import { Card, Avatar, Skeleton } from 'antd';
import { EditOutlined, EllipsisOutlined, SettingOutlined } from '@ant-design/icons';

const { Meta } = Card;

<Card
  style={{ width: 300 }}
  cover={<img alt="cover" src="/image.jpg" />}
  actions={[
    <SettingOutlined key="setting" />,
    <EditOutlined key="edit" />,
    <EllipsisOutlined key="ellipsis" />,
  ]}
>
  <Meta
    avatar={<Avatar src="/avatar.jpg" />}
    title="Card title"
    description="This is the card description"
  />
</Card>

// Loading state
<Card loading={true}>
  <Meta title="Card title" description="Description" />
</Card>
```

### Drawer

```tsx
import { Drawer, Button, Form, Input, Select, Space } from 'antd';

function DrawerForm() {
  const [open, setOpen] = useState(false);

  return (
    <>
      <Button type="primary" onClick={() => setOpen(true)}>
        Create User
      </Button>
      <Drawer
        title="Create a new user"
        width={720}
        open={open}
        onClose={() => setOpen(false)}
        extra={
          <Space>
            <Button onClick={() => setOpen(false)}>Cancel</Button>
            <Button type="primary">Submit</Button>
          </Space>
        }
      >
        <Form layout="vertical">
          <Form.Item label="Name" name="name" rules={[{ required: true }]}>
            <Input placeholder="Enter name" />
          </Form.Item>
          <Form.Item label="Email" name="email">
            <Input placeholder="Enter email" />
          </Form.Item>
        </Form>
      </Drawer>
    </>
  );
}
```

## Theming

### Design Tokens

```tsx
import { ConfigProvider, theme } from 'antd';

// Custom theme
<ConfigProvider
  theme={{
    token: {
      colorPrimary: '#1677ff',
      colorSuccess: '#52c41a',
      colorWarning: '#faad14',
      colorError: '#ff4d4f',
      colorInfo: '#1677ff',
      borderRadius: 6,
      fontFamily: 'Inter, sans-serif',
    },
    components: {
      Button: {
        colorPrimary: '#00b96b',
        algorithm: true,
      },
      Input: {
        colorBorder: '#d9d9d9',
      },
    },
  }}
>
  <App />
</ConfigProvider>

// Dark mode
<ConfigProvider
  theme={{
    algorithm: theme.darkAlgorithm,
  }}
>
  <App />
</ConfigProvider>

// Compact mode
<ConfigProvider
  theme={{
    algorithm: theme.compactAlgorithm,
  }}
>
  <App />
</ConfigProvider>

// Combined algorithms
<ConfigProvider
  theme={{
    algorithm: [theme.darkAlgorithm, theme.compactAlgorithm],
  }}
>
  <App />
</ConfigProvider>
```

### Using Theme Tokens

```tsx
import { theme } from 'antd';

function MyComponent() {
  const { token } = theme.useToken();

  return (
    <div
      style={{
        backgroundColor: token.colorBgContainer,
        color: token.colorText,
        padding: token.padding,
        borderRadius: token.borderRadius,
      }}
    >
      Styled with tokens
    </div>
  );
}
```

## Layout

```tsx
import { Layout, Menu } from 'antd';

const { Header, Sider, Content, Footer } = Layout;

function AppLayout() {
  const [collapsed, setCollapsed] = useState(false);

  return (
    <Layout style={{ minHeight: '100vh' }}>
      <Sider
        collapsible
        collapsed={collapsed}
        onCollapse={setCollapsed}
      >
        <div className="logo" />
        <Menu theme="dark" mode="inline" items={menuItems} />
      </Sider>
      <Layout>
        <Header style={{ padding: 0, background: '#fff' }} />
        <Content style={{ margin: '24px 16px' }}>
          <div style={{ padding: 24, background: '#fff', minHeight: 360 }}>
            Content here
          </div>
        </Content>
        <Footer style={{ textAlign: 'center' }}>
          My App 2024
        </Footer>
      </Layout>
    </Layout>
  );
}
```

## Data Entry

### Select with Search

```tsx
import { Select } from 'antd';

<Select
  showSearch
  placeholder="Select a user"
  optionFilterProp="label"
  options={[
    { value: '1', label: 'John Doe' },
    { value: '2', label: 'Jane Smith' },
  ]}
  filterOption={(input, option) =>
    option.label.toLowerCase().includes(input.toLowerCase())
  }
/>

// Multiple select
<Select
  mode="multiple"
  allowClear
  placeholder="Select tags"
  options={tags}
  maxTagCount="responsive"
/>

// With grouping
<Select
  options={[
    {
      label: 'Manager',
      options: [
        { label: 'Jack', value: 'jack' },
        { label: 'Lucy', value: 'lucy' },
      ],
    },
    {
      label: 'Engineer',
      options: [
        { label: 'Tom', value: 'tom' },
      ],
    },
  ]}
/>
```

### Upload

```tsx
import { Upload, Button } from 'antd';
import { UploadOutlined, InboxOutlined } from '@ant-design/icons';

// Basic upload
<Upload
  action="/api/upload"
  listType="text"
  maxCount={1}
>
  <Button icon={<UploadOutlined />}>Upload File</Button>
</Upload>

// Drag and drop
const { Dragger } = Upload;

<Dragger
  name="file"
  multiple
  action="/api/upload"
  onChange={(info) => {
    if (info.file.status === 'done') {
      message.success(`${info.file.name} uploaded successfully`);
    }
  }}
>
  <p className="ant-upload-drag-icon">
    <InboxOutlined />
  </p>
  <p className="ant-upload-text">Click or drag file to upload</p>
</Dragger>
```

### DatePicker and TimePicker

```tsx
import { DatePicker, TimePicker, Space } from 'antd';
import dayjs from 'dayjs';

const { RangePicker } = DatePicker;

// Single date
<DatePicker
  format="YYYY-MM-DD"
  disabledDate={(current) => current && current < dayjs().startOf('day')}
/>

// Date range
<RangePicker
  format="YYYY-MM-DD"
  presets={[
    { label: 'Last 7 Days', value: [dayjs().subtract(7, 'd'), dayjs()] },
    { label: 'Last 30 Days', value: [dayjs().subtract(30, 'd'), dayjs()] },
  ]}
/>

// Time picker
<TimePicker format="HH:mm" minuteStep={15} />
```

## Best Practices

1. **Use ConfigProvider** - Wrap app for consistent theming
2. **Form.useForm()** - Create form instance for programmatic control
3. **Design tokens** - Use theme tokens instead of hardcoded values
4. **Icons package** - Import from `@ant-design/icons`
5. **Context holders** - Use hook-based APIs for message/notification

## Reference Files

- [references/components.md](references/components.md) - Complete component list
- [references/theming.md](references/theming.md) - Advanced theming patterns
