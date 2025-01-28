//App.js

import React, { useState } from "react";
import { gql, useQuery, useMutation } from "@apollo/client";

// GraphQL Queries and Mutations
const GET_ITEMS = gql`
  query {
    getItems {
      id
      name
      description
    }
  }
`;

const ADD_ITEM = gql`
  mutation AddItem($name: String!, $description: String!) {
    addItem(name: $name, description: $description) {
      id
      name
      description
    }
  }
`;

const UPDATE_ITEM = gql`
  mutation UpdateItem($id: ID!, $name: String!, $description: String!) {
    updateItem(id: $id, name: $name, description: $description) {
      id
      name
      description
    }
  }
`;

const DELETE_ITEM = gql`
  mutation DeleteItem($id: ID!) {
    deleteItem(id: $id)
  }
`;

function App() {
  const { data, loading, error, refetch } = useQuery(GET_ITEMS);
  const [addItem] = useMutation(ADD_ITEM);
  const [updateItem] = useMutation(UPDATE_ITEM);
  const [deleteItem] = useMutation(DELETE_ITEM);

  const [newItem, setNewItem] = useState({ name: "", description: "" });
  const [editItem, setEditItem] = useState({ id: "", name: "", description: "" });

  if (loading) return <p>Loading...</p>;
  if (error) return <p>Error: {error.message}</p>;

  const handleAddItem = async () => {
    await addItem({
      variables: newItem,
    });
    refetch(); // Refetch the items after mutation
    setNewItem({ name: "", description: "" });
  };

  const handleUpdateItem = async () => {
    await updateItem({
      variables: { id: editItem.id, name: editItem.name, description: editItem.description },
    });
    refetch(); // Refetch the items after mutation
    setEditItem({ id: "", name: "", description: "" });
  };

  const handleDeleteItem = async (id) => {
    await deleteItem({
      variables: { id },
    });
    refetch(); // Refetch the items after mutation
  };

  const handleEditClick = (item) => {
    setEditItem({ id: item.id, name: item.name, description: item.description });
  };

  return (
    <div>
      <h1>GraphQL Apollo Client CRUD Example</h1>

      <div>
        <h2>Add Item</h2>
        <input
          type="text"
          placeholder="Name"
          value={newItem.name}
          onChange={(e) => setNewItem({ ...newItem, name: e.target.value })}
        />
        <input
          type="text"
          placeholder="Description"
          value={newItem.description}
          onChange={(e) =>
            setNewItem({ ...newItem, description: e.target.value })
          }
        />
        <button onClick={handleAddItem}>Add Item</button>
      </div>

      <div>
        <h2>Edit Item</h2>
        {editItem.id && (
          <>
            <input
              type="text"
              value={editItem.name}
              onChange={(e) =>
                setEditItem({ ...editItem, name: e.target.value })
              }
            />
            <input
              type="text"
              value={editItem.description}
              onChange={(e) =>
                setEditItem({ ...editItem, description: e.target.value })
              }
            />
            <button onClick={handleUpdateItem}>Update Item</button>
          </>
        )}
      </div>

      <h2>Items</h2>
      <ul>
        {data?.getItems.map((item) => (
          <li key={item.id}>
            <h3>{item.name}</h3>
            <p>{item.description}</p>
            <button onClick={() => handleEditClick(item)}>Edit</button>
            <button onClick={() => handleDeleteItem(item.id)}>Delete</button>
          </li>
        ))}
      </ul>
    </div>
  );
}

export default App;


//main.jsx

import React from "react";
import ReactDOM from "react-dom/client";
import ApolloClientProvider from "./schema";
import App from "./App";

// Render the App component wrapped with Apollo Client Provider
const root = ReactDOM.createRoot(document.getElementById("root"));
root.render(
  <ApolloClientProvider>
    <App />
  </ApolloClientProvider>
);


//schema.jsx

import React from "react";
import { ApolloClient, InMemoryCache, ApolloProvider } from "@apollo/client";

// Apollo Client Setup
const client = new ApolloClient({
  uri: "http://localhost:4000/graphql",  // Use the local server URL
  cache: new InMemoryCache(),
});

// ApolloClientProvider to wrap around the App component
const ApolloClientProvider = ({ children }) => {
  return <ApolloProvider client={client}>{children}</ApolloProvider>;
};

export default ApolloClientProvider;


//server.js

import { ApolloServer, gql } from "apollo-server";

const items = [
  { id: "1", name: "Item 1", description: "Description 1" },
  { id: "2", name: "Item 2", description: "Description 2" },
];

// Type definitions (Schema)
const typeDefs = gql`
  type Item {
    id: ID!
    name: String!
    description: String!
  }

  type Query {
    getItems: [Item]
  }

  type Mutation {
    addItem(name: String!, description: String!): Item
    updateItem(id: ID!, name: String!, description: String!): Item
    deleteItem(id: ID!): String
  }
`;

// Resolvers
const resolvers = {
  Query: {
    getItems: () => items,
  },
  Mutation: {
    addItem: (_, { name, description }) => {
      const newItem = { id: `${items.length + 1}`, name, description };
      items.push(newItem);
      return newItem;
    },
    updateItem: (_, { id, name, description }) => {
      const itemIndex = items.findIndex((item) => item.id === id);
      if (itemIndex > -1) {
        items[itemIndex] = { id, name, description };
        return items[itemIndex];
      }
      throw new Error("Item not found");
    },
    deleteItem: (_, { id }) => {
      const itemIndex = items.findIndex((item) => item.id === id);
      if (itemIndex > -1) {
        items.splice(itemIndex, 1);
        return "Item deleted successfully";
      }
      throw new Error("Item not found");
    },
  },
};

// Apollo Server Setup
const server = new ApolloServer({ typeDefs, resolvers });

server.listen().then(({ url }) => {
  console.log(`Server ready at ${url}`);
});

//package.json
{
  "name": "graphql",
  "private": true,
  "version": "0.0.0",
  "type": "module",
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "lint": "eslint .",
    "preview": "vite preview"
  },
  "dependencies": {
    "@apollo/client": "^3.12.8",
    "apollo-server": "^3.13.0",
    "graphql": "^16.10.0",
    "graphql-tools": "^9.0.11",
    "react": "^18.3.1",
    "react-dom": "^18.3.1"
  },
  "devDependencies": {
    "@eslint/js": "^9.17.0",
    "@types/react": "^18.3.18",
    "@types/react-dom": "^18.3.5",
    "@vitejs/plugin-react": "^4.3.4",
    "eslint": "^9.17.0",
    "eslint-plugin-react": "^7.37.2",
    "eslint-plugin-react-hooks": "^5.0.0",
    "eslint-plugin-react-refresh": "^0.4.16",
    "globals": "^15.14.0",
    "vite": "^6.0.5"
  }
}

