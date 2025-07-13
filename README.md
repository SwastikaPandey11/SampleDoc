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
