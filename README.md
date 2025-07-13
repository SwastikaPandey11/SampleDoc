import React from 'react';
import { FormControl, InputLabel, Select, MenuItem, Box } from '@mui/material';
import { useMQConnection } from '../../../hooks/useMQConnection';

export const MQConnectionForm = () => {
  const {
    queueManagers,
    queues,
    selectedManager,
    selectedQueue,
    loading,
    handleManagerChange,
    handleQueueChange
  } = useMQConnection();

  return (
    <Box display="flex" gap={2} mb={3}>
      <FormControl fullWidth>
        <InputLabel>Queue Manager</InputLabel>
        <Select
          value={selectedManager}
          onChange={handleManagerChange}
          label="Queue Manager"
        >
          {queueManagers.map((qm) => (
            <MenuItem key={qm} value={qm}>{qm}</MenuItem>
          ))}
        </Select>
      </FormControl>
      
      <FormControl fullWidth>
        <InputLabel>Queue</InputLabel>
        <Select
          value={selectedQueue}
          onChange={handleQueueChange}
          label="Queue"
          disabled={!selectedManager || loading}
        >
          {queues.map((q) => (
            <MenuItem key={q} value={q}>{q}</MenuItem>
          ))}
        </Select>
      </FormControl>
    </Box>
  );
};



//useMQConnection.ts
import { useState, useEffect } from 'react';
import { getQueueManagers, getQueues } from '../services/mqService';

export const useMQConnection = () => {
  const [queueManagers, setQueueManagers] = useState<string[]>([]);
  const [queues, setQueues] = useState<string[]>([]);
  const [selectedManager, setSelectedManager] = useState<string>('');
  const [selectedQueue, setSelectedQueue] = useState<string>('');
  const [loading, setLoading] = useState<boolean>(false);

  useEffect(() => {
    const fetchQueueManagers = async () => {
      try {
        const managers = await getQueueManagers();
        setQueueManagers(managers);
      } catch (error) {
        console.error('Failed to fetch queue managers:', error);
      }
    };
    fetchQueueManagers();
  }, []);

  useEffect(() => {
    const fetchQueues = async () => {
      if (!selectedManager) return;
      
      try {
        setLoading(true);
        const queueList = await getQueues(selectedManager);
        setQueues(queueList);
        setSelectedQueue('');
      } catch (error) {
        console.error('Failed to fetch queues:', error);
      } finally {
        setLoading(false);
      }
    };
    fetchQueues();
  }, [selectedManager]);

  const handleManagerChange = (manager: string) => {
    setSelectedManager(manager);
  };

  const handleQueueChange = (queue: string) => {
    setSelectedQueue(queue);
  };

  return {
    queueManagers,
    queues,
    selectedManager,
    selectedQueue,
    loading,
    handleManagerChange,
    handleQueueChange
  };
};




//mqService.ts
import axios from './apiClient';

export const getQueueManagers = async (): Promise<string[]> => {
  const response = await axios.get('/mq/managers');
  return response.data;
};

export const getQueues = async (manager: string): Promise<string[]> => {
  const response = await axios.get('/mq/queues', {
    params: { manager }
  });
  return response.data;
};

export const sendMessage = async (
  manager: string,
  queue: string,
  message: string
): Promise<void> => {
  await axios.post('/mq/send', message, {
    params: { manager, queue },
    headers: { 'Content-Type': 'text/plain' }
  });
};


//homePage
import React from 'react';
import { Box, Button, Paper, Typography } from '@mui/material';
import { MQConnectionForm } from '../../components/MQConnectionForm';
import { MessageEditor } from '../../components/MessageEditor';
import { StatusAlert } from '../../components/StatusAlert';
import { useMessageSender } from '../../hooks/useMessageSender';

export const HomePage = () => {
  const {
    message,
    setMessage,
    error,
    success,
    loading,
    handleSend,
    clearStatus
  } = useMessageSender();

  return (
    <Box>
      <Typography variant="h4" gutterBottom>
        IBM MQ Message Sender
      </Typography>
      
      <Paper elevation={3} sx={{ p: 3, mb: 3 }}>
        <MQConnectionForm />
        
        <MessageEditor 
          message={message}
          onChange={setMessage}
        />
        
        <Button
          variant="contained"
          color="primary"
          onClick={handleSend}
          disabled={loading}
        >
          Send Message
        </Button>
      </Paper>
      
      <StatusAlert 
        error={error}
        success={success}
        onClose={clearStatus}
      />
    </Box>
  );
};




//App.tsx
import React from 'react';
import { CssBaseline, ThemeProvider } from '@mui/material';
import { HomePage } from './pages/HomePage';
import { theme } from './styles/theme';
import { GlobalStyles } from './styles/globalStyles';
import { AppLayout } from './pages/Layout';

const App = () => {
  return (
    <ThemeProvider theme={theme}>
      <CssBaseline />
      <GlobalStyles />
      <AppLayout>
        <HomePage />
      </AppLayout>
    </ThemeProvider>
  );
};

export default App;



styles golbal
import { createTheme } from '@mui/material/styles';

export const theme = createTheme({
  palette: {
    primary: {
      main: '#1976d2',
      contrastText: '#ffffff',
    },
    secondary: {
      main: '#dc004e',
    },
    background: {
      default: '#f5f5f5',
      paper: '#ffffff',
    },
  },
  typography: {
    fontFamily: '"Roboto", "Helvetica", "Arial", sans-serif',
    h4: {
      fontWeight: 600,
    },
  },
  shape: {
    borderRadius: 8,
  },
});



//
