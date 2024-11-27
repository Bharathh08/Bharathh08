// App.js
import React, { useState } from 'react';
import { View, Text, Button, StyleSheet, TextInput, FlatList } from 'react-native';
import firebase, { auth } from './firebase';
import axios from 'axios';

export default function App() {
  const [user, setUser] = useState(null);
  const [servers, setServers] = useState([]);
  const [serverName, setServerName] = useState('');

  const handleGoogleSignIn = async () => {
    const provider = new firebase.auth.GoogleAuthProvider();
    try {
      const result = await auth.signInWithPopup(provider);
      setUser(result.user);
    } catch (error) {
      console.error(error);
    }
  };

  const fetchServers = async () => {
    try {
      const response = await axios.get('http://localhost:5000/getServers');
      setServers(response.data);
    } catch (error) {
      console.error(error);
    }
  };

  const createServer = async () => {
    try {
      await axios.post('http://localhost:5000/createServer', {
        serverName,
        createdBy: user.uid,
      });
      fetchServers();
    } catch (error) {
      console.error(error);
    }
  };

  return (
    <View style={styles.container}>
      {!user ? (
        <Button title="Sign in with Google" onPress={handleGoogleSignIn} />
      ) : (
        <>
          <Text style={styles.header}>Welcome, {user.displayName}</Text>
          <View style={styles.inputContainer}>
            <TextInput
              style={styles.input}
              placeholder="Server Name"
              value={serverName}
              onChangeText={setServerName}
            />
            <Button title="Create Server" onPress={createServer} />
          </View>
          <FlatList
            data={servers}
            keyExtractor={item => item.id}
            renderItem={({ item }) => (
              <View style={styles.serverItem}>
                <Text>{item.serverName}</Text>
              </View>
            )}
          />
        </>
      )}
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    padding: 20,
    backgroundColor: '#f4f4f4',
  },
  header: {
    fontSize: 24,
    fontWeight: 'bold',
    marginBottom: 20,
  },
  inputContainer: {
    flexDirection: 'row',
    marginBottom: 20,
  },
  input: {
    flex: 1,
    borderWidth: 1,
    borderColor: '#ccc',
    borderRadius: 5,
    padding: 10,
    marginRight: 10,
  },
  serverItem: {
    padding: 15,
    borderBottomWidth: 1,
    borderBottomColor: '#ccc',
  },
});
