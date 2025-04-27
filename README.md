App.js
import React, { useState } from 'react';
import { Container, CssBaseline, Typography, Box } from '@mui/material';
import FileUploader from './components/FileUploader';
import ColumnSelector from './components/ColumnSelector';
import DataDisplay from './components/DataDisplay';

function App() {
  const [csvData, setCsvData] = useState(null);
  const [columns, setColumns] = useState([]);
  const [selectedColumns, setSelectedColumns] = useState([]);
  const [displayData, setDisplayData] = useState(null);

  const handleFileUpload = (data, columns) => {
    setCsvData(data);
    setColumns(columns);
    setSelectedColumns([]);
    setDisplayData(null);
  };

  const handleColumnSelect = (selected) => {
    setSelectedColumns(selected);
  };

  const handleSave = () => {
    if (selectedColumns.length === 0) return;
    
    const filteredData = csvData.map(row => {
      const filteredRow = {};
      selectedColumns.forEach(col => {
        filteredRow[col] = row[col];
      });
      return filteredRow;
    });
    
    setDisplayData(filteredData);
  };

  return (
    <>
      <CssBaseline />
      <Container maxWidth="lg" sx={{ py: 4 }}>
        <Typography variant="h3" component="h1" gutterBottom align="center" sx={{ fontWeight: 'bold', mb: 4 }}>
          CSV Data Explorer
        </Typography>
        
        <Box sx={{ 
          backgroundColor: 'background.paper', 
          borderRadius: 2, 
          boxShadow: 1, 
          p: 4, 
          mb: 4 
        }}>
          <FileUploader onFileUpload={handleFileUpload} />
        </Box>

        {columns.length > 0 && (
          <Box sx={{ 
            backgroundColor: 'background.paper', 
            borderRadius: 2, 
            boxShadow: 1, 
            p: 4, 
            mb: 4 
          }}>
            <ColumnSelector 
              columns={columns} 
              onSelect={handleColumnSelect} 
              onSave={handleSave} 
            />
          </Box>
        )}

        {displayData && (
          <Box sx={{ 
            backgroundColor: 'background.paper', 
            borderRadius: 2, 
            boxShadow: 1, 
            p: 4 
          }}>
            <DataDisplay data={displayData} />
          </Box>
        )}
      </Container>
    </>
  );
}

export default App;






FileUploader.js
import React from 'react';
import { Button, Box, Typography } from '@mui/material';
import { Upload as UploadIcon } from '@mui/icons-material';
import Papa from 'papaparse';

const FileUploader = ({ onFileUpload }) => {
  const handleFileChange = (event) => {
    const file = event.target.files[0];
    if (!file) return;

    Papa.parse(file, {
      header: true,
      complete: (results) => {
        const columns = results.meta.fields || [];
        onFileUpload(results.data, columns);
      },
      error: (error) => {
        console.error('Error parsing CSV:', error);
        alert('Error parsing CSV file. Please check the file format.');
      }
    });
  };

  return (
    <Box sx={{ textAlign: 'center' }}>
      <Typography variant="h5" component="h2" gutterBottom>
        Upload CSV File
      </Typography>
      <Typography variant="body1" color="text.secondary" gutterBottom sx={{ mb: 3 }}>
        Select a CSV file to analyze its contents
      </Typography>
      <Button
        variant="contained"
        component="label"
        startIcon={<UploadIcon />}
        size="large"
        sx={{
          py: 2,
          px: 4,
          borderRadius: 2,
          textTransform: 'none',
          fontSize: '1.1rem'
        }}
      >
        Choose File
        <input
          type="file"
          hidden
          accept=".csv"
          onChange={handleFileChange}
        />
      </Button>
    </Box>
  );
};

export default FileUploader;



ColumnSelector.js


import React, { useState, useEffect } from 'react';
import { 
  Box, 
  Typography, 
  List, 
  ListItem, 
  Checkbox, 
  ListItemText, 
  Button,
  Chip,
  Divider
} from '@mui/material';
import { Save as SaveIcon } from '@mui/icons-material';

const ColumnSelector = ({ columns, onSelect, onSave }) => {
  const [selected, setSelected] = useState([]);

  useEffect(() => {
    setSelected([]);
  }, [columns]);

  const handleToggle = (column) => () => {
    const currentIndex = selected.indexOf(column);
    const newSelected = [...selected];

    if (currentIndex === -1) {
      newSelected.push(column);
    } else {
      newSelected.splice(currentIndex, 1);
    }

    setSelected(newSelected);
    onSelect(newSelected);
  };

  return (
    <Box>
      <Typography variant="h5" component="h2" gutterBottom>
        Select Columns to Display
      </Typography>
      <Typography variant="body1" color="text.secondary" gutterBottom sx={{ mb: 3 }}>
        Choose the columns you want to view from the CSV file
      </Typography>

      {selected.length > 0 && (
        <Box sx={{ mb: 3 }}>
          <Typography variant="subtitle1" gutterBottom>
            Selected Columns:
          </Typography>
          <Box sx={{ display: 'flex', flexWrap: 'wrap', gap: 1 }}>
            {selected.map((column) => (
              <Chip 
                key={column} 
                label={column} 
                onDelete={handleToggle(column)} 
                color="primary"
              />
            ))}
          </Box>
        </Box>
      )}

      <Divider sx={{ my: 2 }} />

      <List dense sx={{ 
        maxHeight: 300, 
        overflow: 'auto', 
        border: '1px solid #eee', 
        borderRadius: 1,
        mb: 3
      }}>
        {columns.map((column) => (
          <ListItem
            key={column}
            button
            onClick={handleToggle(column)}
            sx={{
              '&:hover': {
                backgroundColor: 'action.hover'
              }
            }}
          >
            <Checkbox
              edge="start"
              checked={selected.indexOf(column) !== -1}
              tabIndex={-1}
              disableRipple
            />
            <ListItemText primary={column} />
          </ListItem>
        ))}
      </List>

      <Button
        variant="contained"
        color="primary"
        startIcon={<SaveIcon />}
        onClick={onSave}
        disabled={selected.length === 0}
        size="large"
        sx={{
          py: 1.5,
          px: 4,
          borderRadius: 2,
          textTransform: 'none',
          fontSize: '1.1rem'
        }}
      >
        Display Selected Data
      </Button>
    </Box>
  );
};

export default ColumnSelector;



DataDisplay.js
import React from 'react';
import { 
  Box, 
  Typography, 
  Table, 
  TableBody, 
  TableCell, 
  TableContainer, 
  TableHead, 
  TableRow,
  Paper
} from '@mui/material';

const DataDisplay = ({ data }) => {
  if (!data || data.length === 0) return null;

  const columns = Object.keys(data[0]);

  return (
    <Box>
      <Typography variant="h5" component="h2" gutterBottom sx={{ mb: 3 }}>
        Displaying Selected Data
      </Typography>
      
      <TableContainer component={Paper} sx={{ maxHeight: 500, overflow: 'auto' }}>
        <Table stickyHeader>
          <TableHead>
            <TableRow>
              {columns.map((column) => (
                <TableCell key={column} sx={{ fontWeight: 'bold' }}>
                  {column}
                </TableCell>
              ))}
            </TableRow>
          </TableHead>
          <TableBody>
            {data.map((row, index) => (
              <TableRow key={index}>
                {columns.map((column) => (
                  <TableCell key={`${index}-${column}`}>
                    {row[column] || '-'}
                  </TableCell>
                ))}
              </TableRow>
            ))}
          </TableBody>
        </Table>
      </TableContainer>
    </Box>
  );
};

export default DataDisplay;




Index.js

import React from 'react';
import ReactDOM from 'react-dom';
import App from './App';
import { ThemeProvider, createTheme } from '@mui/material/styles';
import CssBaseline from '@mui/material/CssBaseline';

const theme = createTheme({
  palette: {
    mode: 'light',
    primary: {
      main: '#1976d2',
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
  },
});

ReactDOM.render(
  <React.StrictMode>
    <ThemeProvider theme={theme}>
      <CssBaseline />
      <App />
    </ThemeProvider>
  </React.StrictMode>,
  document.getElementById('root')
);
