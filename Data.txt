# Modularized Full-Stack IBM MQ Message Sender Application

I'll restructure the application into clean, modular components for better maintainability and scalability. Here's the improved architecture:

## Frontend Modular Structure

```
src/
├── api/                  # API service layer
│   ├── mqApi.ts          # IBM MQ specific API calls
│   └── index.ts          # API exports
├── components/           # Reusable UI components
│   ├── layout/           # Layout components
│   │   ├── Header.tsx
│   │   └── MainContainer.tsx
│   └── mq/               # IBM MQ specific components
│       ├── QueueSelector.tsx
│       ├── MessageEditor.tsx
│       └── MessageStatusAlert.tsx
├── features/             # Feature-based modules
│   └── messageSender/    # Message sending feature
│       ├── MessageSenderForm.tsx
│       ├── useMessageSender.ts  # Custom hook for business logic
│       └── types.ts      # Type definitions
├── pages/                # Page components
│   └── HomePage.tsx      # Main application page
├── App.tsx               # Main app component
└── index.tsx             # Entry point
```

## Backend Modular Structure

```
src/main/java/com/example/mqsender/
├── config/               # Configuration classes
│   ├── MqConfig.java     # IBM MQ configuration
│   └── WebConfig.java    # Web/API configuration
├── controller/           # API controllers
│   ├── MqManagerController.java
│   ├── MqQueueController.java
│   └── MqMessageController.java
├── model/                # Data models
│   ├── dto/              # DTOs for API communication
│   ├── entity/           # MongoDB entities
│   └── enums/            # Enumerations
├── repository/           # Data repositories
│   ├── MqManagerRepository.java
│   └── MqQueueRepository.java
├── service/              # Business logic
│   ├── impl/             # Service implementations
│   └── MqService.java    # Service interface
└── exception/            # Exception handling
    ├── MqException.java
    └── GlobalExceptionHandler.java
```

## Updated Frontend Implementation

### 1. API Layer (`src/api/mqApi.ts`)

```typescript
import axios from 'axios';

const api = axios.create({
  baseURL: process.env.REACT_APP_API_BASE_URL || 'http://localhost:8080/api',
});

export interface QueueManager {
  id: string;
  name: string;
  host: string;
  port: number;
  channel: string;
}

export interface Queue {
  id: string;
  name: string;
  managerId: string;
}

export interface SendMessageRequest {
  queueManagerId: string;
  queueId: string;
  message: string;
}

export const mqApi = {
  getQueueManagers: async (): Promise<QueueManager[]> => {
    const response = await api.get('/managers');
    return response.data;
  },
  
  getQueues: async (): Promise<Queue[]> => {
    const response = await api.get('/queues');
    return response.data;
  },
  
  sendMessage: async (request: SendMessageRequest): Promise<void> => {
    await api.post('/messages', request);
  }
};
```

### 2. Custom Hook (`src/features/messageSender/useMessageSender.ts`)

```typescript
import { useState, useEffect } from 'react';
import { mqApi, QueueManager, Queue } from '../../api/mqApi';

export const useMessageSender = () => {
  const [queueManagers, setQueueManagers] = useState<QueueManager[]>([]);
  const [queues, setQueues] = useState<Queue[]>([]);
  const [isLoading, setIsLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);
  
  useEffect(() => {
    const fetchData = async () => {
      try {
        setIsLoading(true);
        const [managers, queues] = await Promise.all([
          mqApi.getQueueManagers(),
          mqApi.getQueues()
        ]);
        setQueueManagers(managers);
        setQueues(queues);
      } catch (err) {
        setError('Failed to load queue data');
      } finally {
        setIsLoading(false);
      }
    };
    
    fetchData();
  }, []);
  
  const sendMessage = async (request: {
    queueManagerId: string;
    queueId: string;
    message: string;
  }): Promise<void> => {
    try {
      setIsLoading(true);
      setError(null);
      await mqApi.sendMessage(request);
    } catch (err) {
      setError('Failed to send message');
      throw err;
    } finally {
      setIsLoading(false);
    }
  };
  
  return {
    queueManagers,
    queues,
    isLoading,
    error,
    sendMessage
  };
};
```

### 3. Queue Selector Component (`src/components/mq/QueueSelector.tsx`)

```typescript
import React from 'react';
import { FormControl, InputLabel, Select, MenuItem, SelectChangeEvent } from '@mui/material';

interface QueueSelectorProps {
  label: string;
  value: string;
  options: Array<{ id: string; name: string }>;
  onChange: (value: string) => void;
  disabled?: boolean;
}

export const QueueSelector: React.FC<QueueSelectorProps> = ({
  label,
  value,
  options,
  onChange,
  disabled = false
}) => {
  const handleChange = (event: SelectChangeEvent) => {
    onChange(event.target.value as string);
  };

  return (
    <FormControl fullWidth>
      <InputLabel>{label}</InputLabel>
      <Select
        value={value}
        label={label}
        onChange={handleChange}
        disabled={disabled}
      >
        {options.map((option) => (
          <MenuItem key={option.id} value={option.id}>
            {option.name}
          </MenuItem>
        ))}
      </Select>
    </FormControl>
  );
};
```

### 4. Message Sender Form (`src/features/messageSender/MessageSenderForm.tsx`)

```typescript
import React, { useState } from 'react';
import { Button, TextField, Box, Typography } from '@mui/material';
import SendIcon from '@mui/icons-material/Send';
import { QueueSelector } from '../../components/mq/QueueSelector';
import { MessageStatusAlert } from '../../components/mq/MessageStatusAlert';
import { useMessageSender } from './useMessageSender';

export const MessageSenderForm: React.FC = () => {
  const { queueManagers, queues, isLoading, error, sendMessage } = useMessageSender();
  const [message, setMessage] = useState('');
  const [selectedQueueManager, setSelectedQueueManager] = useState('');
  const [selectedQueue, setSelectedQueue] = useState('');
  const [success, setSuccess] = useState<string | null>(null);

  const filteredQueues = queues.filter(q => q.managerId === selectedQueueManager);

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    setSuccess(null);
    
    try {
      await sendMessage({
        queueManagerId: selectedQueueManager,
        queueId: selectedQueue,
        message
      });
      setSuccess('Message sent successfully!');
      setMessage('');
    } catch (err) {
      // Error is already handled in the hook
    }
  };

  return (
    <Box component="form" onSubmit={handleSubmit} sx={{ mt: 3 }}>
      <Typography variant="h5" gutterBottom>
        Send Message to IBM MQ
      </Typography>
      
      <MessageStatusAlert 
        error={error}
        success={success}
        isLoading={isLoading}
      />
      
      <QueueSelector
        label="Queue Manager"
        value={selectedQueueManager}
        options={queueManagers.map(qm => ({
          id: qm.id,
          name: `${qm.name} (${qm.host}:${qm.port})`
        }))}
        onChange={setSelectedQueueManager}
      />
      
      <QueueSelector
        label="Queue"
        value={selectedQueue}
        options={filteredQueues.map(q => ({
          id: q.id,
          name: q.name
        }))}
        onChange={setSelectedQueue}
        disabled={!selectedQueueManager}
        sx={{ mt: 2 }}
      />
      
      <TextField
        label="Message"
        multiline
        rows={6}
        fullWidth
        value={message}
        onChange={(e) => setMessage(e.target.value)}
        sx={{ mt: 2, mb: 2 }}
        required
      />
      
      <Button
        type="submit"
        variant="contained"
        size="large"
        endIcon={<SendIcon />}
        disabled={isLoading || !selectedQueueManager || !selectedQueue || !message}
      >
        Send Message
      </Button>
    </Box>
  );
};
```

## Updated Backend Implementation

### 1. DTOs (`src/main/java/com/example/mqsender/model/dto`)

`QueueManagerDto.java`:
```java
package com.example.mqsender.model.dto;

import lombok.Data;

@Data
public class QueueManagerDto {
    private String id;
    private String name;
    private String host;
    private int port;
    private String channel;
}
```

`QueueDto.java`:
```java
package com.example.mqsender.model.dto;

import lombok.Data;

@Data
public class QueueDto {
    private String id;
    private String name;
    private String managerId;
}
```

`SendMessageRequest.java`:
```java
package com.example.mqsender.model.dto;

import lombok.Data;

@Data
public class SendMessageRequest {
    private String queueManagerId;
    private String queueId;
    private String message;
}
```

### 2. Service Interface (`src/main/java/com/example/mqsender/service/MqService.java`)

```java
package com.example.mqsender.service;

import com.example.mqsender.model.dto.QueueDto;
import com.example.mqsender.model.dto.QueueManagerDto;
import com.example.mqsender.model.dto.SendMessageRequest;

import java.util.List;

public interface MqService {
    List<QueueManagerDto> getAllQueueManagers();
    List<QueueDto> getAllQueues();
    List<QueueDto> getQueuesByManager(String managerId);
    void sendMessage(SendMessageRequest request);
}
```

### 3. Service Implementation (`src/main/java/com/example/mqsender/service/impl/MqServiceImpl.java`)

```java
package com.example.mqsender.service.impl;

import com.example.mqsender.model.dto.QueueDto;
import com.example.mqsender.model.dto.QueueManagerDto;
import com.example.mqsender.model.dto.SendMessageRequest;
import com.example.mqsender.model.entity.QueueEntity;
import com.example.mqsender.model.entity.QueueManagerEntity;
import com.example.mqsender.repository.QueueManagerRepository;
import com.example.mqsender.repository.QueueRepository;
import com.example.mqsender.service.MqService;
import com.ibm.mq.jms.MQQueueConnectionFactory;
import lombok.RequiredArgsConstructor;
import org.modelmapper.ModelMapper;
import org.springframework.jms.core.JmsTemplate;
import org.springframework.stereotype.Service;

import javax.jms.JMSException;
import java.util.List;
import java.util.stream.Collectors;

@Service
@RequiredArgsConstructor
public class MqServiceImpl implements MqService {
    
    private final QueueManagerRepository queueManagerRepository;
    private final QueueRepository queueRepository;
    private final MQQueueConnectionFactory mqQueueConnectionFactory;
    private final JmsTemplate jmsTemplate;
    private final ModelMapper modelMapper;
    
    @Override
    public List<QueueManagerDto> getAllQueueManagers() {
        return queueManagerRepository.findAll().stream()
            .map(this::convertToManagerDto)
            .collect(Collectors.toList());
    }
    
    @Override
    public List<QueueDto> getAllQueues() {
        return queueRepository.findAll().stream()
            .map(this::convertToQueueDto)
            .collect(Collectors.toList());
    }
    
    @Override
    public List<QueueDto> getQueuesByManager(String managerId) {
        return queueRepository.findByManagerId(managerId).stream()
            .map(this::convertToQueueDto)
            .collect(Collectors.toList());
    }
    
    @Override
    public void sendMessage(SendMessageRequest request) {
        var manager = queueManagerRepository.findById(request.getQueueManagerId())
            .orElseThrow(() -> new RuntimeException("Queue manager not found"));
        
        var queue = queueRepository.findById(request.getQueueId())
            .orElseThrow(() -> new RuntimeException("Queue not found"));
        
        if (!queue.getManagerId().equals(manager.getId())) {
            throw new RuntimeException("Queue does not belong to the specified manager");
        }
        
        try {
            configureConnectionFactory(manager);
            jmsTemplate.convertAndSend(queue.getName(), request.getMessage());
        } catch (Exception e) {
            throw new RuntimeException("Failed to send message to IBM MQ", e);
        }
    }
    
    private void configureConnectionFactory(QueueManagerEntity manager) throws JMSException {
        mqQueueConnectionFactory.setHostName(manager.getHost());
        mqQueueConnectionFactory.setPort(manager.getPort());
        mqQueueConnectionFactory.setQueueManager(manager.getName());
        mqQueueConnectionFactory.setChannel(manager.getChannel());
    }
    
    private QueueManagerDto convertToManagerDto(QueueManagerEntity entity) {
        return modelMapper.map(entity, QueueManagerDto.class);
    }
    
    private QueueDto convertToQueueDto(QueueEntity entity) {
        return modelMapper.map(entity, QueueDto.class);
    }
}
```

### 4. Controllers

`MqManagerController.java`:
```java
package com.example.mqsender.controller;

import com.example.mqsender.model.dto.QueueManagerDto;
import com.example.mqsender.service.MqService;
import lombok.RequiredArgsConstructor;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.List;

@RestController
@RequestMapping("/api/managers")
@RequiredArgsConstructor
public class MqManagerController {
    
    private final MqService mqService;
    
    @GetMapping
    public List<QueueManagerDto> getAllQueueManagers() {
        return mqService.getAllQueueManagers();
    }
}
```

`MqQueueController.java`:
```java
package com.example.mqsender.controller;

import com.example.mqsender.model.dto.QueueDto;
import com.example.mqsender.service.MqService;
import lombok.RequiredArgsConstructor;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

import java.util.List;

@RestController
@RequestMapping("/api/queues")
@RequiredArgsConstructor
public class MqQueueController {
    
    private final MqService mqService;
    
    @GetMapping
    public List<QueueDto> getAllQueues() {
        return mqService.getAllQueues();
    }
    
    @GetMapping("/by-manager")
    public List<QueueDto> getQueuesByManager(@RequestParam String managerId) {
        return mqService.getQueuesByManager(managerId);
    }
}
```

`MqMessageController.java`:
```java
package com.example.mqsender.controller;

import com.example.mqsender.model.dto.SendMessageRequest;
import com.example.mqsender.service.MqService;
import lombok.RequiredArgsConstructor;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/api/messages")
@RequiredArgsConstructor
public class MqMessageController {
    
    private final MqService mqService;
    
    @PostMapping
    public void sendMessage(@RequestBody SendMessageRequest request) {
        mqService.sendMessage(request);
    }
}
```

## Key Improvements in Modularization

1. **Frontend**:
   - Separated API layer with TypeScript interfaces
   - Feature-based organization with custom hooks
   - Reusable UI components
   - Clear separation of concerns between presentation and logic

2. **Backend**:
   - Proper layered architecture (Controller -> Service -> Repository)
   - DTOs for API communication
   - Interface-based service layer
   - Modular controllers by resource type
   - ModelMapper for entity-DTO conversion

3. **Both**:
   - Consistent naming conventions
   - Single responsibility principle applied
   - Clear module boundaries
   - Easier to test and maintain

This modular structure makes the application more maintainable, testable, and scalable. Each component has a clear responsibility, and dependencies are properly managed through interfaces and dependency injection.
