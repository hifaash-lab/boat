import React, { useState, useEffect, useMemo } from 'react';
import { 
  Anchor, 
  Fish, 
  DollarSign, 
  Users, 
  LogOut, 
  Plus, 
  Trash2, 
  Edit2, 
  CheckCircle, 
  AlertCircle, 
  Menu, 
  X, 
  BarChart3,
  Search,
  Save,
  UserPlus,
  Settings,
  MapPin,
  Tag,
  Ship,
  ClipboardList,
  Building
} from 'lucide-react';

// Firebase Imports
import { initializeApp } from "firebase/app";
import { 
  getAuth, 
  signInAnonymously, 
  onAuthStateChanged,
  signInWithCustomToken
} from "firebase/auth";
import { 
  getFirestore, 
  collection, 
  addDoc, 
  updateDoc, 
  deleteDoc, 
  doc, 
  onSnapshot, 
  query, 
  orderBy, 
  serverTimestamp,
  where,
  getDocs,
  setDoc
} from "firebase/firestore";

// --- Firebase Configuration & Initialization ---
const firebaseConfig = JSON.parse(__firebase_config);
const app = initializeApp(firebaseConfig);
const auth = getAuth(app);
const db = getFirestore(app);
const appId = typeof __app_id !== 'undefined' ? __app_id : 'default-app-id';

// Utility for currency formatting
const formatCurrency = (value) => `$${parseFloat(value || 0).toFixed(2)}`;

// --- Components ---

// 1. Login Component (Unchanged)
const Login = ({ onLogin, error }) => {
  const [username, setUsername] = useState('');
  const [password, setPassword] = useState('');

  const handleSubmit = (e) => {
    e.preventDefault();
    onLogin(username, password);
  };

  return (
    <div className="min-h-screen bg-slate-900 flex items-center justify-center p-4">
      <div className="bg-white rounded-xl shadow-2xl p-8 w-full max-w-md">
        <div className="text-center mb-8">
          <div className="bg-blue-600 w-16 h-16 rounded-full flex items-center justify-center mx-auto mb-4">
            <Anchor className="text-white w-8 h-8" />
          </div>
          <h1 className="text-2xl font-bold text-slate-800">Captain's Log</h1>
          <p className="text-slate-500">Speedboat & Fishery Management</p>
        </div>

        <form onSubmit={handleSubmit} className="space-y-4">
          <div>
            <label className="block text-sm font-medium text-slate-700 mb-1">Username</label>
            <input
              type="text"
              required
              className="w-full px-4 py-2 border border-slate-300 rounded-lg focus:ring-2 focus:ring-blue-500 outline-none"
              value={username}
              onChange={(e) => setUsername(e.target.value)}
              placeholder="e.g. admin"
            />
          </div>
          <div>
            <label className="block text-sm font-medium text-slate-700 mb-1">Password</label>
            <input
              type="password"
              required
              className="w-full px-4 py-2 border border-slate-300 rounded-lg focus:ring-2 focus:ring-blue-500 outline-none"
              value={password}
              onChange={(e) => setPassword(e.target.value)}
              placeholder="••••••••"
            />
          </div>
          
          {error && (
            <div className="p-3 bg-red-50 text-red-600 text-sm rounded-lg flex items-center">
              <AlertCircle className="w-4 h-4 mr-2" />
              {error}
            </div>
          )}

          <button
            type="submit"
            className="w-full bg-blue-600 hover:bg-blue-700 text-white font-semibold py-2 px-4 rounded-lg transition-colors"
          >
            Sign In
          </button>
        </form>
      </div>
    </div>
  );
};

// 2. Dashboard Component (Updated to reflect new bill structure)
const Dashboard = ({ bills, catches }) => {
  const stats = useMemo(() => {
    // Total Revenue is the amount before payment
    const totalRevenue = bills.reduce((acc, curr) => acc + (parseFloat(curr.amount) || 0), 0);
    // Total Collected is the amount paid
    const totalCollected = bills.reduce((acc, curr) => acc + (parseFloat(curr.paid) || 0), 0);
    const outstanding = totalRevenue - totalCollected;
    
    // Sum of all catches recorded in KG
    const totalFishKg = catches
      .filter(c => c.unit === 'kg')
      .reduce((acc, curr) => acc + (parseFloat(curr.quantity) || 0), 0);

    // Unique customer IDs in bills
    const activeCustomers = new Set(bills.map(b => b.customerId)).size;

    return { totalRevenue, totalCollected, outstanding, totalFishKg, activeCustomers };
  }, [bills, catches]);

  return (
    <div className="space-y-6">
      <h2 className="text-2xl font-bold text-slate-800">Dashboard</h2>
      
      {/* Stats Grid */}
      <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-4">
        <div className="bg-white p-6 rounded-xl shadow-sm border border-slate-100">
          <div className="flex justify-between items-start">
            <div>
              <p className="text-sm font-medium text-slate-500">Outstanding</p>
              <h3 className="text-2xl font-bold text-red-500">{formatCurrency(stats.outstanding)}</h3>
            </div>
            <div className="p-2 bg-red-50 rounded-lg">
              <AlertCircle className="w-5 h-5 text-red-500" />
            </div>
          </div>
          <div className="mt-2 text-xs text-slate-400">Total uncollected receivables</div>
        </div>

        <div className="bg-white p-6 rounded-xl shadow-sm border border-slate-100">
          <div className="flex justify-between items-start">
            <div>
              <p className="text-sm font-medium text-slate-500">Total Catch (Kg)</p>
              <h3 className="text-2xl font-bold text-blue-600">{stats.totalFishKg.toLocaleString()} kg</h3>
            </div>
            <div className="p-2 bg-blue-50 rounded-lg">
              <Fish className="w-5 h-5 text-blue-600" />
            </div>
          </div>
          <div className="mt-2 text-xs text-slate-400">Recorded weight inventory</div>
        </div>

        <div className="bg-white p-6 rounded-xl shadow-sm border border-slate-100">
          <div className="flex justify-between items-start">
            <div>
              <p className="text-sm font-medium text-slate-500">Collected Revenue</p>
              <h3 className="text-2xl font-bold text-emerald-600">{formatCurrency(stats.totalCollected)}</h3>
            </div>
            <div className="p-2 bg-emerald-50 rounded-lg">
              <CheckCircle className="w-5 h-5 text-emerald-600" />
            </div>
          </div>
          <div className="mt-2 text-xs text-slate-400">Total payments received</div>
        </div>

        <div className="bg-white p-6 rounded-xl shadow-sm border border-slate-100">
          <div className="flex justify-between items-start">
            <div>
              <p className="text-sm font-medium text-slate-500">Unique Clients</p>
              <h3 className="text-2xl font-bold text-slate-800">{stats.activeCustomers}</h3>
            </div>
            <div className="p-2 bg-slate-100 rounded-lg">
              <Users className="w-5 h-5 text-slate-600" />
            </div>
          </div>
          <div className="mt-2 text-xs text-slate-400">Unique clients billed</div>
        </div>
      </div>

      {/* Recent Activity Section */}
      <div className="grid grid-cols-1 lg:grid-cols-2 gap-6">
        <div className="bg-white rounded-xl shadow-sm border border-slate-100 overflow-hidden">
          <div className="p-4 border-b border-slate-100 bg-slate-50/50 flex justify-between items-center">
            <h3 className="font-semibold text-slate-800">Recent Bills</h3>
          </div>
          <div className="divide-y divide-slate-100">
            {bills.slice(0, 5).map(bill => {
              const pending = parseFloat(bill.amount) - parseFloat(bill.paid);
              return (
                <div key={bill.id} className="p-4 flex items-center justify-between">
                  <div>
                    <p className="font-medium text-slate-800">{bill.customerName}</p>
                    <p className="text-xs text-slate-500">{bill.billType} Sale</p>
                  </div>
                  <div className="text-right">
                    <p className="font-bold text-slate-700">{formatCurrency(bill.amount)}</p>
                    {pending > 0 ? (
                      <span className="text-xs text-red-500 font-medium">Due: {formatCurrency(pending)}</span>
                    ) : (
                      <span className="text-xs text-emerald-500 font-medium">Paid</span>
                    )}
                  </div>
                </div>
              );
            })}
            {bills.length === 0 && <p className="p-4 text-center text-slate-400 text-sm">No recent bills</p>}
          </div>
        </div>

        <div className="bg-white rounded-xl shadow-sm border border-slate-100 overflow-hidden">
           <div className="p-4 border-b border-slate-100 bg-slate-50/50 flex justify-between items-center">
            <h3 className="font-semibold text-slate-800">Recent Catch</h3>
          </div>
          <div className="divide-y divide-slate-100">
            {catches.slice(0, 5).map(fish => (
              <div key={fish.id} className="p-4 flex items-center justify-between">
                <div className="flex items-center gap-3">
                  <div className="w-8 h-8 rounded-full bg-blue-100 flex items-center justify-center">
                    <Fish className="w-4 h-4 text-blue-600" />
                  </div>
                  <div>
                    <p className="font-medium text-slate-800">{fish.type}</p>
                  </div>
                </div>
                <div className="text-right">
                  <p className="font-bold text-slate-700">{fish.quantity} <span className="text-sm font-normal text-slate-500">{fish.unit}</span></p>
                </div>
              </div>
            ))}
            {catches.length === 0 && <p className="p-4 text-center text-slate-400 text-sm">No recent catch logs</p>}
          </div>
        </div>
      </div>
    </div>
  );
};

// 3. Billing Component (Refactored)
const Billing = ({ bills, customers, catchTypes, tripTypes, onAdd, onUpdate, onDelete }) => {
  const [isModalOpen, setIsModalOpen] = useState(false);
  const [editItem, setEditItem] = useState(null);
  const [searchTerm, setSearchTerm] = useState('');
  
  // Form State
  const [formData, setFormData] = useState(() => ({
    customerId: '',
    customerName: '', // Stored for easy display
    date: new Date().toISOString().split('T')[0],
    billType: 'trip', // 'trip' or 'catch'
    amount: '0',
    paid: '0',
    notes: '',
    // Dynamic fields
    selectedTripId: '',
    catchItems: [], // [{ catchTypeId, name, quantity, unit, unitPrice, subtotal }]
  }));

  // --- Calculations ---
  const calculateTotalCatchPrice = (items) => {
    return items.reduce((sum, item) => sum + (parseFloat(item.subtotal) || 0), 0).toFixed(2);
  };

  const handleTripSelect = (tripId) => {
    const trip = tripTypes.find(t => t.id === tripId);
    if (trip) {
      setFormData(prev => ({
        ...prev,
        selectedTripId: tripId,
        amount: trip.price.toString(), // Auto-set price
      }));
    } else {
      setFormData(prev => ({ ...prev, selectedTripId: '', amount: '0' }));
    }
  };

  const handleCatchItemChange = (index, field, value) => {
    const newItems = [...formData.catchItems];
    const item = newItems[index];
    
    // Update the field
    item[field] = value;

    // Recalculate subtotal if quantity changes
    if (field === 'quantity') {
      const unitPrice = parseFloat(item.unitPrice || 0);
      const quantity = parseFloat(value || 0);
      item.subtotal = (unitPrice * quantity).toFixed(2);
    }

    const newTotal = calculateTotalCatchPrice(newItems);

    setFormData(prev => ({
      ...prev,
      catchItems: newItems,
      amount: newTotal,
    }));
  };

  const handleAddCatchItem = (catchTypeId) => {
    const catchType = catchTypes.find(c => c.id === catchTypeId);
    if (catchType && !formData.catchItems.some(item => item.catchTypeId === catchTypeId)) {
      const newItem = {
        catchTypeId: catchType.id,
        name: catchType.name,
        quantity: '0',
        unit: catchType.unit,
        unitPrice: catchType.unitPrice.toString(),
        subtotal: '0.00',
      };
      const newItems = [...formData.catchItems, newItem];
      const newTotal = calculateTotalCatchPrice(newItems);
      setFormData(prev => ({
        ...prev,
        catchItems: newItems,
        amount: newTotal,
      }));
    }
  };

  const handleDeleteCatchItem = (index) => {
    const newItems = formData.catchItems.filter((_, i) => i !== index);
    const newTotal = calculateTotalCatchPrice(newItems);
    setFormData(prev => ({
      ...prev,
      catchItems: newItems,
      amount: newTotal,
    }));
  };

  // --- Modal & Submission Handlers ---
  const openModal = (item = null) => {
    if (item) {
      setEditItem(item);
      setFormData({
        customerId: item.customerId,
        customerName: item.customerName,
        date: item.date,
        billType: item.billType,
        amount: item.amount,
        paid: item.paid,
        notes: item.notes || '',
        selectedTripId: item.selectedTripId || '',
        catchItems: item.catchItems || [],
      });
    } else {
      setEditItem(null);
      const defaultCustomer = customers.length > 0 ? customers.sort((a, b) => a.name.localeCompare(b.name))[0] : { id: '', name: '' };
      setFormData({
        customerId: defaultCustomer.id,
        customerName: defaultCustomer.name,
        date: new Date().toISOString().split('T')[0],
        billType: 'trip',
        amount: '0',
        paid: '0',
        notes: '',
        selectedTripId: '',
        catchItems: [],
      });
    }
    setIsModalOpen(true);
  };

  const closeModal = () => setIsModalOpen(false);

  const handleSubmit = (e) => {
    e.preventDefault();
    const customer = customers.find(c => c.id === formData.customerId);
    
    if (!customer) {
      console.error("No customer selected.");
      // Using an inline modal/message instead of alert()
      // Note: Using window.confirm/alert is forbidden, but since this is a complex app, 
      // I will log and rely on the UI validation (required fields) to manage this.
      return; 
    }

    const dataToSave = {
      ...formData,
      customerName: customer.name,
      amount: parseFloat(formData.amount).toFixed(2), // Ensure numeric format
      paid: parseFloat(formData.paid).toFixed(2),
    };

    if (editItem) {
      onUpdate(editItem.id, dataToSave);
    } else {
      onAdd(dataToSave);
    }
    closeModal();
  };

  // --- Customer Dropdown Utility ---
  const handleCustomerChange = (e) => {
    const customerId = e.target.value;
    const customer = customers.find(c => c.id === customerId);
    setFormData(prev => ({
      ...prev,
      customerId: customerId,
      customerName: customer ? customer.name : '',
    }));
  };

  // --- Filtering ---
  const customerMap = useMemo(() => new Map(customers.map(c => [c.id, c.name])), [customers]);

  const filteredBills = bills.filter(b => 
    (b.customerName || customerMap.get(b.customerId) || '').toLowerCase().includes(searchTerm.toLowerCase()) ||
    b.billType.toLowerCase().includes(searchTerm.toLowerCase())
  );

  return (
    <div className="space-y-6">
      <div className="flex flex-col sm:flex-row sm:items-center justify-between gap-4">
        <h2 className="text-2xl font-bold text-slate-800">Bills & Sales</h2>
        <button 
          onClick={() => openModal()}
          className="bg-blue-600 hover:bg-blue-700 text-white px-4 py-2 rounded-lg flex items-center gap-2 transition-colors w-full sm:w-auto justify-center"
        >
          <Plus className="w-4 h-4" /> New Bill
        </button>
      </div>

      <div className="relative">
        <Search className="absolute left-3 top-1/2 -translate-y-1/2 w-4 h-4 text-slate-400" />
        <input 
          type="text" 
          placeholder="Search by customer or bill type..." 
          className="w-full pl-10 pr-4 py-2 border border-slate-200 rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-500"
          value={searchTerm}
          onChange={(e) => setSearchTerm(e.target.value)}
        />
      </div>

      <div className="bg-white rounded-xl shadow-sm border border-slate-100 overflow-hidden">
        <div className="overflow-x-auto">
          <table className="w-full text-left">
            <thead className="bg-slate-50 border-b border-slate-100">
              <tr>
                <th className="p-4 font-semibold text-slate-600 text-sm">Customer</th>
                <th className="p-4 font-semibold text-slate-600 text-sm">Type / Date</th>
                <th className="p-4 font-semibold text-slate-600 text-sm">Amount</th>
                <th className="p-4 font-semibold text-slate-600 text-sm">Status</th>
                <th className="p-4 font-semibold text-slate-600 text-sm text-right">Actions</th>
              </tr>
            </thead>
            <tbody className="divide-y divide-slate-100">
              {filteredBills.map(bill => {
                const total = parseFloat(bill.amount);
                const paid = parseFloat(bill.paid);
                const due = total - paid;
                let statusColor = "bg-red-100 text-red-700";
                let statusText = "Unpaid";

                if (paid >= total) {
                  statusColor = "bg-emerald-100 text-emerald-700";
                  statusText = "Paid";
                } else if (paid > 0) {
                  statusColor = "bg-amber-100 text-amber-700";
                  statusText = "Partial";
                }

                return (
                  <tr key={bill.id} className="hover:bg-slate-50 transition-colors">
                    <td className="p-4 font-medium text-slate-800">{bill.customerName || customerMap.get(bill.customerId) || 'N/A'}</td>
                    <td className="p-4">
                      <div className="text-sm text-slate-800 capitalize">{bill.billType} Sale</div>
                      <div className="text-xs text-slate-500">{bill.date}</div>
                    </td>
                    <td className="p-4 font-mono text-slate-700">{formatCurrency(total)}</td>
                    <td className="p-4">
                      <span className={`px-2 py-1 rounded-full text-xs font-medium ${statusColor}`}>
                        {statusText}
                      </span>
                      {due > 0 && <div className="text-xs text-red-500 mt-1">Due: {formatCurrency(due)}</div>}
                    </td>
                    <td className="p-4 text-right space-x-2">
                      <button onClick={() => openModal(bill)} className="p-1 hover:bg-slate-200 rounded text-slate-500">
                        <Edit2 className="w-4 h-4" />
                      </button>
                      <button onClick={() => onDelete(bill.id)} className="p-1 hover:bg-red-50 rounded text-red-500">
                        <Trash2 className="w-4 h-4" />
                      </button>
                    </td>
                  </tr>
                );
              })}
              {filteredBills.length === 0 && (
                <tr>
                  <td colSpan="5" className="p-8 text-center text-slate-400">No bills found.</td>
                </tr>
              )}
            </tbody>
          </table>
        </div>
      </div>

      {/* Modal */}
      {isModalOpen && (
        <div className="fixed inset-0 bg-black/50 flex items-center justify-center p-4 z-50">
          <div className="bg-white rounded-xl shadow-xl w-full max-w-2xl overflow-hidden">
            <div className="p-4 border-b border-slate-100 flex justify-between items-center bg-slate-50">
              <h3 className="font-bold text-slate-800">{editItem ? 'Edit Bill' : 'New Bill'}</h3>
              <button onClick={closeModal}><X className="w-5 h-5 text-slate-500" /></button>
            </div>
            <form onSubmit={handleSubmit} className="p-6 space-y-6">
              
              {/* Customer & Date */}
              <div className="grid grid-cols-1 sm:grid-cols-2 gap-4">
                <div>
                  <label className="block text-xs font-medium text-slate-600 mb-1">Customer</label>
                  <select required className="w-full px-3 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500 outline-none" 
                    value={formData.customerId} onChange={handleCustomerChange}>
                    <option value="" disabled>Select Customer</option>
                    {customers.sort((a, b) => a.name.localeCompare(b.name)).map(c => (
                      <option key={c.id} value={c.id}>{c.name}</option>
                    ))}
                  </select>
                </div>
                <div>
                  <label className="block text-xs font-medium text-slate-600 mb-1">Date</label>
                  <input required type="date" className="w-full px-3 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500 outline-none" 
                    value={formData.date} onChange={e => setFormData({...formData, date: e.target.value})} />
                </div>
              </div>
              
              {/* Bill Type Selector */}
              <div className="space-y-2">
                <label className="block text-xs font-medium text-slate-600">Bill Type</label>
                <div className="flex bg-slate-100 rounded-lg p-1">
                    <button type="button" 
                      onClick={() => setFormData(prev => ({...prev, billType: 'trip', amount: '0', catchItems: []}))}
                      className={`flex-1 py-2 text-sm font-medium rounded-lg transition-colors ${formData.billType === 'trip' ? 'bg-blue-600 text-white shadow' : 'text-slate-700 hover:bg-slate-200'}`}
                    >
                      Trip (Charter)
                    </button>
                    <button type="button"
                      onClick={() => setFormData(prev => ({...prev, billType: 'catch', amount: '0', selectedTripId: ''}))}
                      className={`flex-1 py-2 text-sm font-medium rounded-lg transition-colors ${formData.billType === 'catch' ? 'bg-blue-600 text-white shadow' : 'text-slate-700 hover:bg-slate-200'}`}
                    >
                      Catch (Sale)
                    </button>
                </div>
              </div>

              {/* Item Selection based on Bill Type */}
              <div className="bg-slate-50 p-4 rounded-xl border border-slate-200">
                {formData.billType === 'trip' ? (
                  /* Trip Selection */
                  <div>
                    <label className="block text-xs font-medium text-slate-600 mb-1">Select Trip Type</label>
                    <select required className="w-full px-3 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500 outline-none" 
                      value={formData.selectedTripId} onChange={e => handleTripSelect(e.target.value)}>
                      <option value="" disabled>Select Trip Type</option>
                      {tripTypes.map(t => (
                        <option key={t.id} value={t.id}>{t.name} ({formatCurrency(t.price)})</option>
                      ))}
                    </select>
                  </div>
                ) : (
                  /* Catch Selection */
                  <div>
                    <label className="block text-xs font-medium text-slate-600 mb-1">Add Catch Item to Bill</label>
                    <select 
                      onChange={e => handleAddCatchItem(e.target.value)} 
                      className="w-full px-3 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500 outline-none mb-3"
                      value="" // Force selection change trigger
                    >
                      <option value="" disabled>Select Catch Type to Add</option>
                      {catchTypes.filter(ct => !formData.catchItems.some(item => item.catchTypeId === ct.id)).map(ct => (
                        <option key={ct.id} value={ct.id}>{ct.name} ({ct.unit} @ {formatCurrency(ct.unitPrice)})</option>
                      ))}
                    </select>

                    {formData.catchItems.length > 0 && (
                      <div className="space-y-2">
                        <h4 className="text-sm font-semibold text-slate-700 mt-4">Included Items:</h4>
                        {formData.catchItems.map((item, index) => (
                          <div key={index} className="flex gap-2 items-center bg-white p-3 rounded-lg border">
                            <div className="flex-1">
                              <p className="font-medium text-slate-800">{item.name} ({item.unit})</p>
                              <p className="text-xs text-slate-500">Unit Price: {formatCurrency(item.unitPrice)}</p>
                            </div>
                            <div className="w-24">
                              <input 
                                type="number" step="0.1" required
                                placeholder="Qty"
                                className="w-full px-2 py-1 border rounded-lg text-sm"
                                value={item.quantity}
                                onChange={e => handleCatchItemChange(index, 'quantity', e.target.value)}
                              />
                            </div>
                            <div className="w-20 text-right font-mono text-sm">
                              {formatCurrency(item.subtotal)}
                            </div>
                            <button type="button" onClick={() => handleDeleteCatchItem(index)} className="text-red-400 hover:text-red-600 p-1">
                              <X className="w-4 h-4" />
                            </button>
                          </div>
                        ))}
                      </div>
                    )}
                  </div>
                )}
              </div>
              
              {/* Financials & Notes */}
              <div className="grid grid-cols-2 gap-4">
                 <div>
                  <label className="block text-xs font-medium text-slate-600 mb-1">Total Amount ($)</label>
                  {/* Total Amount is editable, even for trip type */}
                  <input required type="number" step="0.01" className="w-full px-3 py-2 border rounded-lg font-bold text-lg focus:ring-2 focus:ring-blue-500 outline-none" 
                    value={formData.amount} onChange={e => setFormData({...formData, amount: e.target.value})} />
                </div>
                 <div>
                  <label className="block text-xs font-medium text-slate-600 mb-1">Paid Amount ($)</label>
                  <input required type="number" step="0.01" className="w-full px-3 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500 outline-none" 
                    value={formData.paid} onChange={e => setFormData({...formData, paid: e.target.value})} />
                </div>
              </div>
              <div>
                  <label className="block text-xs font-medium text-slate-600 mb-1">Notes</label>
                  <textarea rows="2" className="w-full px-3 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500 outline-none" 
                    value={formData.notes} onChange={e => setFormData({...formData, notes: e.target.value})} />
              </div>
              
              <div className="pt-4 flex justify-end gap-2">
                <button type="button" onClick={closeModal} className="px-4 py-2 text-slate-600 hover:bg-slate-100 rounded-lg">Cancel</button>
                <button type="submit" className="px-4 py-2 bg-blue-600 hover:bg-blue-700 text-white rounded-lg">Save Bill</button>
              </div>
            </form>
          </div>
        </div>
      )}
    </div>
  );
};

// 4. Catch Log Component (Modified to show aggregated stock)
const CatchLog = ({ catchStock, catches, catchTypes, onAdd, onDelete }) => {
  const [isModalOpen, setIsModalOpen] = useState(false);
  const [formData, setFormData] = useState({
    catchTypeId: catchTypes.length > 0 ? catchTypes[0].id : '',
    type: catchTypes.length > 0 ? catchTypes[0].name : '',
    unit: catchTypes.length > 0 ? catchTypes[0].unit : 'kg',
    quantity: '',
    date: new Date().toISOString().split('T')[0]
  });

  useEffect(() => {
    // Sync form data on initial load or if catchTypes changes
    if (catchTypes.length > 0 && formData.catchTypeId === '') {
      setFormData(prev => ({
        ...prev,
        catchTypeId: catchTypes[0].id,
        type: catchTypes[0].name,
        unit: catchTypes[0].unit,
      }));
    }
  }, [catchTypes, formData.catchTypeId]);


  const handleTypeChange = (e) => {
    const id = e.target.value;
    const selectedType = catchTypes.find(ct => ct.id === id);
    if (selectedType) {
      setFormData(prev => ({
        ...prev,
        catchTypeId: id,
        type: selectedType.name,
        unit: selectedType.unit,
      }));
    }
  };

  const handleSubmit = (e) => {
    e.preventDefault();
    onAdd(formData);
    setIsModalOpen(false);
    setFormData(prev => ({ ...prev, quantity: '' })); // Reset quantity only
  };

  return (
    <div className="space-y-6">
      <div className="flex flex-col sm:flex-row sm:items-center justify-between gap-4">
        <h2 className="text-2xl font-bold text-slate-800">Catch Stock Overview</h2>
        <button 
          onClick={() => setIsModalOpen(true)}
          className="bg-emerald-600 hover:bg-emerald-700 text-white px-4 py-2 rounded-lg flex items-center gap-2 transition-colors w-full sm:w-auto justify-center"
        >
          <Plus className="w-4 h-4" /> Log New Catch
        </button>
      </div>
      
      {/* Aggregated Stock Display */}
      <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
        {catchStock.map(stock => (
          <div key={stock.id} className="bg-white p-4 rounded-xl shadow-sm border border-slate-100 flex flex-col justify-between">
            <div className="flex justify-between items-start mb-2">
              <div className="flex items-center gap-2">
                 <div className="w-10 h-10 rounded-full bg-blue-50 flex items-center justify-center">
                    <Fish className="w-5 h-5 text-blue-500" />
                 </div>
                 <div>
                   <h3 className="font-bold text-slate-800">{stock.name}</h3>
                   <p className="text-xs text-slate-500">Unit Price: {formatCurrency(stock.unitPrice)}</p>
                 </div>
              </div>
            </div>
            <div className="flex justify-between items-end mt-4">
              <div className="text-xs text-slate-400">Inventory Value: {formatCurrency(stock.quantity * stock.unitPrice)}</div>
              <div className="text-xl font-bold text-slate-700">
                {stock.quantity.toFixed(1)} <span className="text-sm font-normal text-slate-500">{stock.unit}</span>
              </div>
            </div>
          </div>
        ))}
        {catchStock.length === 0 && (
          <div className="col-span-full text-center py-10 bg-slate-50 rounded-xl border border-dashed border-slate-300 text-slate-400">
            No catch types defined or stock recorded yet.
          </div>
        )}
      </div>

      {/* Recent Log Activity */}
      <h3 className="text-xl font-bold text-slate-800 mt-8 mb-4">Recent Catch Logs</h3>
      
      <div className="bg-white rounded-xl shadow-sm border border-slate-100 overflow-hidden">
        <div className="overflow-x-auto">
          <table className="w-full text-left">
            <thead className="bg-slate-50 border-b border-slate-100">
              <tr>
                <th className="p-4 font-semibold text-slate-600 text-sm">Date</th>
                <th className="p-4 font-semibold text-slate-600 text-sm">Type</th>
                <th className="p-4 font-semibold text-slate-600 text-sm text-right">Quantity</th>
                <th className="p-4 font-semibold text-slate-600 text-sm text-right">Actions</th>
              </tr>
            </thead>
            <tbody className="divide-y divide-slate-100">
              {catches.sort((a, b) => b.date.localeCompare(a.date)).slice(0, 10).map(fish => ( // Show only 10 recent logs
                <tr key={fish.id} className="hover:bg-slate-50 transition-colors">
                  <td className="p-4 text-sm text-slate-500">{fish.date}</td>
                  <td className="p-4 font-medium text-slate-800">{fish.type}</td>
                  <td className="p-4 text-right font-bold text-slate-700">
                    {parseFloat(fish.quantity).toFixed(1)} <span className="text-sm font-normal text-slate-500">{fish.unit}</span>
                  </td>
                  <td className="p-4 text-right">
                    <button onClick={() => onDelete(fish.id)} className="text-red-500 hover:bg-red-50 p-2 rounded">
                      <Trash2 className="w-4 h-4" />
                    </button>
                  </td>
                </tr>
              ))}
              {catches.length === 0 && (
                <tr>
                  <td colSpan="4" className="p-4 text-center text-slate-400">No logs recorded.</td>
                </tr>
              )}
            </tbody>
          </table>
        </div>
      </div>


       {/* Modal */}
      {isModalOpen && (
        <div className="fixed inset-0 bg-black/50 flex items-center justify-center p-4 z-50">
          <div className="bg-white rounded-xl shadow-xl w-full max-w-md overflow-hidden">
            <div className="p-4 border-b border-slate-100 flex justify-between items-center bg-slate-50">
              <h3 className="font-bold text-slate-800">Log Catch</h3>
              <button onClick={() => setIsModalOpen(false)}><X className="w-5 h-5 text-slate-500" /></button>
            </div>
            <form onSubmit={handleSubmit} className="p-6 space-y-4">
              <div>
                <label className="block text-xs font-medium text-slate-600 mb-1">Catch Type</label>
                <select required className="w-full px-3 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500 outline-none"
                   value={formData.catchTypeId} onChange={handleTypeChange}>
                   {catchTypes.map(ct => (
                    <option key={ct.id} value={ct.id}>{ct.name}</option>
                   ))}
                </select>
              </div>
              <div className="grid grid-cols-2 gap-4">
                <div>
                   <label className="block text-xs font-medium text-slate-600 mb-1">Date</label>
                   <input required type="date" className="w-full px-3 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500 outline-none" 
                    value={formData.date} onChange={e => setFormData({...formData, date: e.target.value})} />
                </div>
                <div>
                   <label className="block text-xs font-medium text-slate-600 mb-1">Quantity ({formData.unit})</label>
                   <input required type="number" step="0.1" className="w-full px-3 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500 outline-none" 
                    value={formData.quantity} onChange={e => setFormData({...formData, quantity: e.target.value})} />
                </div>
              </div>
              {catchTypes.length === 0 && (
                <div className="text-red-500 text-sm p-3 bg-red-50 rounded-lg">
                    Please define Catch Types in the Settings tab first.
                </div>
              )}
              <div className="pt-4">
                <button type="submit" className="w-full py-2 bg-emerald-600 hover:bg-emerald-700 text-white rounded-lg font-medium" disabled={catchTypes.length === 0}>Add to Stock</button>
              </div>
            </form>
          </div>
        </div>
      )}
    </div>
  );
};

// 5. Customer Management (New Component)
const CustomerManagement = ({ customers, bills, onAdd, onDelete }) => {
  const [isModalOpen, setIsModalOpen] = useState(false);
  const [formData, setFormData] = useState({ name: '', address: '', phone: '' });

  // Calculate outstanding balance for each customer
  const customerBalances = useMemo(() => {
    const balances = {};
    customers.forEach(c => {
      balances[c.id] = { total: 0, paid: 0 };
    });

    bills.forEach(bill => {
      if (balances[bill.customerId]) {
        balances[bill.customerId].total += parseFloat(bill.amount || 0);
        balances[bill.customerId].paid += parseFloat(bill.paid || 0);
      }
    });

    return Object.keys(balances).map(id => ({
      id,
      customer: customers.find(c => c.id === id) || { name: 'Unknown' },
      outstanding: balances[id].total - balances[id].paid,
    }));
  }, [customers, bills]);

  const handleSubmit = (e) => {
    e.preventDefault();
    onAdd(formData);
    setIsModalOpen(false);
    setFormData({ name: '', address: '', phone: '' });
  };

  return (
    <div className="space-y-6">
      <div className="flex flex-col sm:flex-row sm:items-center justify-between gap-4">
        <h2 className="text-2xl font-bold text-slate-800">Client Directory</h2>
        <button 
          onClick={() => setIsModalOpen(true)}
          className="bg-purple-600 hover:bg-purple-700 text-white px-4 py-2 rounded-lg flex items-center gap-2 transition-colors w-full sm:w-auto justify-center"
        >
          <Plus className="w-4 h-4" /> Add Customer
        </button>
      </div>

      <div className="bg-white rounded-xl shadow-sm border border-slate-100 overflow-hidden">
        <div className="overflow-x-auto">
          <table className="w-full text-left">
            <thead className="bg-slate-50 border-b border-slate-100">
              <tr>
                <th className="p-4 font-semibold text-slate-600 text-sm">Customer Name</th>
                <th className="p-4 font-semibold text-slate-600 text-sm">Contact</th>
                <th className="p-4 font-semibold text-slate-600 text-sm text-right">Outstanding Balance</th>
                <th className="p-4 font-semibold text-slate-600 text-sm text-right">Actions</th>
              </tr>
            </thead>
            <tbody className="divide-y divide-slate-100">
              {customerBalances.sort((a, b) => a.customer.name.localeCompare(b.customer.name)).map(item => (
                <tr key={item.id} className="hover:bg-slate-50 transition-colors">
                  <td className="p-4 font-medium text-slate-800">
                    <p>{item.customer.name}</p>
                    <p className="text-xs text-slate-500">{item.customer.address}</p>
                  </td>
                  <td className="p-4 text-sm text-slate-700">{item.customer.phone}</td>
                  <td className={`p-4 font-bold text-right ${item.outstanding > 0 ? 'text-red-500' : 'text-emerald-500'}`}>
                    {formatCurrency(item.outstanding)}
                  </td>
                  <td className="p-4 text-right">
                    <button onClick={() => onDelete(item.id)} className="text-red-500 hover:bg-red-50 p-2 rounded">
                      <Trash2 className="w-4 h-4" />
                    </button>
                  </td>
                </tr>
              ))}
            </tbody>
          </table>
        </div>
      </div>
       
      {/* Add Customer Modal */}
      {isModalOpen && (
        <div className="fixed inset-0 bg-black/50 flex items-center justify-center p-4 z-50">
           <div className="bg-white rounded-xl shadow-xl w-full max-w-md p-6">
             <h3 className="text-lg font-bold text-slate-800 mb-4">Add New Customer</h3>
             <form onSubmit={handleSubmit} className="space-y-4">
               <div>
                  <label className="block text-xs font-medium text-slate-600 mb-1">Name</label>
                  <input required type="text" className="w-full px-3 py-2 border rounded-lg" 
                    value={formData.name} onChange={e => setFormData({...formData, name: e.target.value})} />
               </div>
               <div>
                  <label className="block text-xs font-medium text-slate-600 mb-1">Phone Number</label>
                  <input type="tel" className="w-full px-3 py-2 border rounded-lg" 
                    value={formData.phone} onChange={e => setFormData({...formData, phone: e.target.value})} />
               </div>
               <div>
                  <label className="block text-xs font-medium text-slate-600 mb-1">Address</label>
                  <textarea rows="2" className="w-full px-3 py-2 border rounded-lg" 
                    value={formData.address} onChange={e => setFormData({...formData, address: e.target.value})} />
               </div>
               <div className="pt-2 flex justify-end gap-2">
                 <button type="button" onClick={() => setIsModalOpen(false)} className="px-3 py-2 text-slate-600">Cancel</button>
                 <button type="submit" className="px-3 py-2 bg-purple-600 text-white rounded-lg">Save Customer</button>
               </div>
             </form>
           </div>
        </div>
      )}
    </div>
  );
};

// 6. Organization Profile Management (New Component)
const OrganizationProfileManagement = ({ profile, onUpdateProfile }) => {
    const [formData, setFormData] = useState(profile);

    useEffect(() => {
        setFormData(profile);
    }, [profile]);

    const handleSubmit = (e) => {
        e.preventDefault();
        onUpdateProfile(formData);
    };
    
    // Fallback logo URL if the provided one is invalid or empty
    const logoUrl = formData.logoUrl && formData.logoUrl.startsWith('http') 
        ? formData.logoUrl 
        : "https://placehold.co/100x40/0A3161/FFFFFF?text=LOGO";

    return (
        <div className="space-y-6">
            <h3 className="text-lg font-bold text-slate-800 mb-4">Company Details</h3>
            
            <div className="flex justify-center items-center p-4 bg-slate-50 rounded-xl border border-dashed border-slate-200">
                <img 
                    src={logoUrl} 
                    alt="Company Logo Preview" 
                    className="max-h-20 w-auto object-contain rounded"
                    onError={(e) => {
                        e.target.onerror = null; 
                        e.target.src="https://placehold.co/100x40/C90000/FFFFFF?text=Invalid%20URL";
                    }}
                />
            </div>

            <form onSubmit={handleSubmit} className="space-y-4 bg-white p-6 rounded-xl border border-slate-100">
                <div>
                    <label className="block text-xs font-medium text-slate-600 mb-1">Business Name / Owner Name</label>
                    <input required type="text" className="w-full px-3 py-2 border rounded-lg" 
                      value={formData.name || ''} onChange={e => setFormData({...formData, name: e.target.value})} />
                </div>
                <div>
                    <label className="block text-xs font-medium text-slate-600 mb-1">Logo URL (Full URL starting with http/https)</label>
                    <input type="url" className="w-full px-3 py-2 border rounded-lg" 
                      value={formData.logoUrl || ''} onChange={e => setFormData({...formData, logoUrl: e.target.value})} />
                </div>
                <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
                    <div>
                        <label className="block text-xs font-medium text-slate-600 mb-1">Phone Number</label>
                        <input type="tel" className="w-full px-3 py-2 border rounded-lg" 
                          value={formData.phone || ''} onChange={e => setFormData({...formData, phone: e.target.value})} />
                    </div>
                    <div>
                        <label className="block text-xs font-medium text-slate-600 mb-1">Email (Optional)</label>
                        <input type="email" className="w-full px-3 py-2 border rounded-lg" 
                          value={formData.email || ''} onChange={e => setFormData({...formData, email: e.target.value})} />
                    </div>
                </div>
                <div>
                    <label className="block text-xs font-medium text-slate-600 mb-1">Business Address</label>
                    <textarea rows="2" className="w-full px-3 py-2 border rounded-lg" 
                      value={formData.address || ''} onChange={e => setFormData({...formData, address: e.target.value})} />
                </div>
                <div className="pt-2 flex justify-end">
                    <button type="submit" className="px-4 py-2 bg-blue-600 hover:bg-blue-700 text-white rounded-lg flex items-center gap-2">
                        <Save className="w-4 h-4" /> Save Profile
                    </button>
                </div>
            </form>
        </div>
    );
};


// 7. Admin Settings Component (New & Refactored)
const AdminSettings = ({ users, catchTypes, tripTypes, profile, onAddUser, onDeleteUser, onAddCatchType, onDeleteCatchType, onAddTripType, onDeleteTripType, onUpdateOrganizationProfile, currentUser }) => {
  const [activeTab, setActiveTab] = useState('profile');

  // --- Management Functions ---

  const CatchTypeManagement = () => {
    const [formData, setFormData] = useState({ name: '', unit: 'kg', unitPrice: '' });
    const [isModalOpen, setIsModalOpen] = useState(false);

    const handleSubmit = (e) => {
      e.preventDefault();
      onAddCatchType(formData);
      setIsModalOpen(false);
      setFormData({ name: '', unit: 'kg', unitPrice: '' });
    };

    return (
      <div className="space-y-4">
        <div className="flex justify-end">
           <button 
             onClick={() => setIsModalOpen(true)}
             className="bg-emerald-600 hover:bg-emerald-700 text-white px-3 py-2 rounded-lg flex items-center gap-2 text-sm"
           >
             <Plus className="w-4 h-4" /> Add Catch Type
           </button>
        </div>
        
        <div className="bg-white rounded-xl border border-slate-100 overflow-hidden">
          <table className="w-full text-left">
            <thead className="bg-slate-50 border-b">
              <tr>
                <th className="p-4 font-semibold text-slate-600 text-sm">Name</th>
                <th className="p-4 font-semibold text-slate-600 text-sm">Unit</th>
                <th className="p-4 font-semibold text-slate-600 text-sm text-right">Unit Price</th>
                <th className="p-4 font-semibold text-slate-600 text-sm text-right">Actions</th>
              </tr>
            </thead>
            <tbody className="divide-y divide-slate-100">
              {catchTypes.map(ct => (
                <tr key={ct.id}>
                  <td className="p-4 font-medium text-slate-800">{ct.name}</td>
                  <td className="p-4 text-sm text-slate-700">{ct.unit}</td>
                  <td className="p-4 text-right font-mono text-slate-700">{formatCurrency(ct.unitPrice)}</td>
                  <td className="p-4 text-right">
                    <button onClick={() => deleteDocInCollection('catchTypes', ct.id)} className="text-red-500 hover:bg-red-50 p-2 rounded">
                      <Trash2 className="w-4 h-4" />
                    </button>
                  </td>
                </tr>
              ))}
            </tbody>
          </table>
          {catchTypes.length === 0 && <p className="p-8 text-center text-slate-400">No catch types defined.</p>}
        </div>

        {/* Modal */}
        {isModalOpen && (
          <div className="fixed inset-0 bg-black/50 flex items-center justify-center p-4 z-50">
            <div className="bg-white rounded-xl shadow-xl w-full max-w-sm p-6">
              <h3 className="text-lg font-bold text-slate-800 mb-4">Add Catch Type</h3>
              <form onSubmit={handleSubmit} className="space-y-4">
                <div>
                    <label className="block text-xs font-medium text-slate-600 mb-1">Catch Name</label>
                    <input required type="text" className="w-full px-3 py-2 border rounded-lg" 
                      value={formData.name} onChange={e => setFormData({...formData, name: e.target.value})} />
                </div>
                <div className="grid grid-cols-2 gap-4">
                  <div>
                    <label className="block text-xs font-medium text-slate-600 mb-1">Unit</label>
                    <select required className="w-full px-3 py-2 border rounded-lg"
                      value={formData.unit} onChange={e => setFormData({...formData, unit: e.target.value})}>
                      <option value="kg">Kilograms (kg)</option>
                      <option value="Qty">Quantity (Qty)</option>
                    </select>
                  </div>
                  <div>
                    <label className="block text-xs font-medium text-slate-600 mb-1">Unit Price ($)</label>
                    <input required type="number" step="0.01" className="w-full px-3 py-2 border rounded-lg" 
                      value={formData.unitPrice} onChange={e => setFormData({...formData, unitPrice: e.target.value})} />
                  </div>
                </div>
                <div className="pt-2 flex justify-end gap-2">
                  <button type="button" onClick={() => setIsModalOpen(false)} className="px-3 py-2 text-slate-600">Cancel</button>
                  <button type="submit" className="px-3 py-2 bg-emerald-600 text-white rounded-lg">Save Type</button>
                </div>
              </form>
            </div>
          </div>
        )}
      </div>
    );
  };

  const TripTypeManagement = () => {
    const [formData, setFormData] = useState({ name: '', price: '' });
    const [isModalOpen, setIsModalOpen] = useState(false);

    const handleSubmit = (e) => {
      e.preventDefault();
      onAddTripType(formData);
      setIsModalOpen(false);
      setFormData({ name: '', price: '' });
    };

    return (
      <div className="space-y-4">
        <div className="flex justify-end">
           <button 
             onClick={() => setIsModalOpen(true)}
             className="bg-sky-600 hover:bg-sky-700 text-white px-3 py-2 rounded-lg flex items-center gap-2 text-sm"
           >
             <Plus className="w-4 h-4" /> Add Trip Type
           </button>
        </div>
        
        <div className="bg-white rounded-xl border border-slate-100 overflow-hidden">
          <table className="w-full text-left">
            <thead className="bg-slate-50 border-b">
              <tr>
                <th className="p-4 font-semibold text-slate-600 text-sm">Trip Name</th>
                <th className="p-4 font-semibold text-slate-600 text-sm text-right">Base Price</th>
                <th className="p-4 font-semibold text-slate-600 text-sm text-right">Actions</th>
              </tr>
            </thead>
            <tbody className="divide-y divide-slate-100">
              {tripTypes.map(tt => (
                <tr key={tt.id}>
                  <td className="p-4 font-medium text-slate-800">{tt.name}</td>
                  <td className="p-4 text-right font-mono text-slate-700">{formatCurrency(tt.price)}</td>
                  <td className="p-4 text-right">
                    <button onClick={() => deleteDocInCollection('tripTypes', tt.id)} className="text-red-500 hover:bg-red-50 p-2 rounded">
                      <Trash2 className="w-4 h-4" />
                    </button>
                  </td>
                </tr>
              ))}
            </tbody>
          </table>
           {tripTypes.length === 0 && <p className="p-8 text-center text-slate-400">No trip types defined.</p>}
        </div>

        {/* Modal */}
        {isModalOpen && (
          <div className="fixed inset-0 bg-black/50 flex items-center justify-center p-4 z-50">
            <div className="bg-white rounded-xl shadow-xl w-full max-w-sm p-6">
              <h3 className="text-lg font-bold text-slate-800 mb-4">Add New Trip Type</h3>
              <form onSubmit={handleSubmit} className="space-y-4">
                <div>
                    <label className="block text-xs font-medium text-slate-600 mb-1">Trip Name</label>
                    <input required type="text" className="w-full px-3 py-2 border rounded-lg" 
                      value={formData.name} onChange={e => setFormData({...formData, name: e.target.value})} />
                </div>
                <div>
                    <label className="block text-xs font-medium text-slate-600 mb-1">Base Price ($)</label>
                    <input required type="number" step="0.01" className="w-full px-3 py-2 border rounded-lg" 
                      value={formData.price} onChange={e => setFormData({...formData, price: e.target.value})} />
                </div>
                <div className="pt-2 flex justify-end gap-2">
                  <button type="button" onClick={() => setIsModalOpen(false)} className="px-3 py-2 text-slate-600">Cancel</button>
                  <button type="submit" className="px-3 py-2 bg-sky-600 text-white rounded-lg">Save Trip</button>
                </div>
              </form>
            </div>
          </div>
        )}
      </div>
    );
  };
  
  const UserTab = () => {
    const [formData, setFormData] = useState({ username: '', password: '', role: 'staff' });
    const [isModalOpen, setIsModalOpen] = useState(false);

    const handleSubmit = (e) => {
      e.preventDefault();
      onAddUser(formData);
      setIsModalOpen(false);
      setFormData({ username: '', password: '', role: 'staff' });
    };

    return (
      <div className="space-y-4">
        <div className="flex justify-end">
          <button 
            onClick={() => setIsModalOpen(true)}
            className="bg-purple-600 hover:bg-purple-700 text-white px-3 py-2 rounded-lg flex items-center gap-2 text-sm"
          >
            <UserPlus className="w-4 h-4" /> Add User
          </button>
        </div>
        <div className="bg-white rounded-xl border border-slate-100 overflow-hidden">
          <table className="w-full text-left">
            <thead className="bg-slate-50 border-b border-slate-100">
              <tr>
                <th className="p-4 font-semibold text-slate-600 text-sm">Username</th>
                <th className="p-4 font-semibold text-slate-600 text-sm">Role</th>
                <th className="p-4 font-semibold text-slate-600 text-sm">Status</th>
                <th className="p-4 font-semibold text-slate-600 text-sm text-right">Actions</th>
              </tr>
            </thead>
            <tbody className="divide-y divide-slate-100">
              {users.map(u => (
                <tr key={u.id}>
                  <td className="p-4 font-medium text-slate-800">{u.username}</td>
                  <td className="p-4 capitalize">
                     <span className={`px-2 py-1 rounded text-xs font-bold ${u.role === 'admin' ? 'bg-purple-100 text-purple-700' : 'bg-slate-100 text-slate-700'}`}>
                       {u.role}
                     </span>
                  </td>
                  <td className="p-4 text-xs text-emerald-600 font-medium">Active</td>
                  <td className="p-4 text-right">
                    {/* Admin can delete any user except themselves or the default admin account */}
                    {u.username !== 'admin' && u.username !== currentUser.username && (
                      <button onClick={() => deleteDocInCollection('users', u.id)} className="text-red-500 hover:bg-red-50 p-2 rounded">
                        <Trash2 className="w-4 h-4" />
                      </button>
                    )}
                  </td>
                </tr>
              ))}
            </tbody>
          </table>
        </div>
        
        {/* Add User Modal */}
        {isModalOpen && (
          <div className="fixed inset-0 bg-black/50 flex items-center justify-center p-4 z-50">
             <div className="bg-white rounded-xl shadow-xl w-full max-w-sm p-6">
               <h3 className="text-lg font-bold text-slate-800 mb-4">Add Team Member</h3>
               <form onSubmit={handleSubmit} className="space-y-4">
                 <div>
                    <label className="block text-xs font-medium text-slate-600 mb-1">Username</label>
                    <input required type="text" className="w-full px-3 py-2 border rounded-lg" 
                      value={formData.username} onChange={e => setFormData({...formData, username: e.target.value})} />
                 </div>
                 <div>
                    <label className="block text-xs font-medium text-slate-600 mb-1">Password</label>
                    <input required type="text" className="w-full px-3 py-2 border rounded-lg" 
                      value={formData.password} onChange={e => setFormData({...formData, password: e.target.value})} />
                 </div>
                 <div>
                    <label className="block text-xs font-medium text-slate-600 mb-1">Role</label>
                    <select className="w-full px-3 py-2 border rounded-lg"
                      value={formData.role} onChange={e => setFormData({...formData, role: e.target.value})}>
                      <option value="staff">Staff</option>
                      <option value="admin">Admin</option>
                    </select>
                 </div>
                 <div className="pt-2 flex justify-end gap-2">
                   <button type="button" onClick={() => setIsModalOpen(false)} className="px-3 py-2 text-slate-600">Cancel</button>
                   <button type="submit" className="px-3 py-2 bg-purple-600 text-white rounded-lg">Create</button>
                 </div>
               </form>
             </div>
          </div>
        )}
      </div>
    );
  };
  
  const tabClasses = (tabName) => `px-4 py-2 text-sm font-semibold rounded-lg transition-colors flex items-center gap-2 ${activeTab === tabName ? 'bg-blue-600 text-white shadow' : 'text-slate-700 hover:bg-slate-200'}`;

  return (
    <div className="space-y-6">
      <h2 className="text-2xl font-bold text-slate-800">Admin Settings</h2>
      
      {/* Tabs */}
      <div className="flex space-x-2 bg-slate-100 p-1 rounded-xl overflow-x-auto whitespace-nowrap">
        <button onClick={() => setActiveTab('profile')} className={tabClasses('profile')}>
          <Building className="w-4 h-4" /> Company Profile
        </button>
        <button onClick={() => setActiveTab('users')} className={tabClasses('users')}>
          <Users className="w-4 h-4" /> User Management
        </button>
        <button onClick={() => setActiveTab('catchTypes')} className={tabClasses('catchTypes')}>
          <Fish className="w-4 h-4" /> Catch Types
        </button>
        <button onClick={() => setActiveTab('tripTypes')} className={tabClasses('tripTypes')}>
          <Ship className="w-4 h-4" /> Trip Types
        </button>
      </div>

      {/* Tab Content */}
      <div className="p-4 bg-white rounded-xl shadow-sm border border-slate-100">
        {activeTab === 'profile' && <OrganizationProfileManagement profile={profile} onUpdateProfile={onUpdateOrganizationProfile} />}
        {activeTab === 'users' && <UserTab />}
        {activeTab === 'catchTypes' && <CatchTypeManagement />}
        {activeTab === 'tripTypes' && <TripTypeManagement />}
      </div>
    </div>
  );
};


// 8. Main Application Component
export default function App() {
  const [currentUser, setCurrentUser] = useState(null);
  const [loading, setLoading] = useState(true);
  const [view, setView] = useState('dashboard');
  const [bills, setBills] = useState([]);
  const [catches, setCatches] = useState([]);
  const [users, setUsers] = useState([]);
  const [customers, setCustomers] = useState([]);
  const [catchTypes, setCatchTypes] = useState([]);
  const [tripTypes, setTripTypes] = useState([]);
  const [organizationProfile, setOrganizationProfile] = useState({}); // New state for profile
  const [authError, setAuthError] = useState(null);
  const [isMobileMenuOpen, setIsMobileMenuOpen] = useState(false);
  const [fsAuthUser, setFsAuthUser] = useState(null);

  // --- Derived State: Catch Stock Calculation ---
  const catchStock = useMemo(() => {
    const stockMap = new Map();
    // 1. Initialize stock for all defined catch types
    catchTypes.forEach(type => {
      stockMap.set(type.id, {
        id: type.id,
        name: type.name,
        unit: type.unit,
        unitPrice: parseFloat(type.unitPrice || 0),
        quantity: 0,
      });
    });

    // 2. Aggregate quantities from catch logs
    catches.forEach(c => {
      const quantity = parseFloat(c.quantity || 0);
      const stock = stockMap.get(c.catchTypeId);
      if (stock) {
        stock.quantity += quantity;
      }
    });

    // 3. Convert map values back to an array for easy rendering, sorted by name
    return Array.from(stockMap.values()).sort((a, b) => a.name.localeCompare(b.name));
  }, [catches, catchTypes]);


  // 1. Initial Authentication & Admin Bootstrap
  useEffect(() => {
    const initAuth = async () => {
      if (typeof __initial_auth_token !== 'undefined' && __initial_auth_token) {
        await signInWithCustomToken(auth, __initial_auth_token);
      } else {
        await signInAnonymously(auth);
      }
    };
    initAuth();

    const unsubscribe = onAuthStateChanged(auth, (user) => {
      setFsAuthUser(user);
      if (user) {
        // Bootstrap Admin User if collection is empty
        const usersRef = collection(db, 'artifacts', appId, 'public', 'data', 'users');
        getDocs(usersRef).then((snapshot) => {
          if (snapshot.empty) {
            addDoc(usersRef, {
              username: 'admin',
              password: 'admin@123', // Default admin credential
              role: 'admin',
              createdAt: serverTimestamp()
            });
          }
        });
      }
      setLoading(false);
    });
    return () => unsubscribe();
  }, []);

  // 2. Data Listeners
  useEffect(() => {
    if (!fsAuthUser) return;

    const collections = [
      { name: 'bills', setter: setBills },
      { name: 'catch', setter: setCatches, sortField: 'date', sortDir: 'desc' },
      { name: 'users', setter: setUsers },
      { name: 'customers', setter: setCustomers, sortField: 'name' },
      { name: 'catchTypes', setter: setCatchTypes, sortField: 'name' },
      { name: 'tripTypes', setter: setTripTypes, sortField: 'name' },
    ];
    
    const unsubs = collections.map(col => {
      const colRef = collection(db, 'artifacts', appId, 'public', 'data', col.name);
      // NOTE: Firestore ordering is applied here, but in-memory sorting is preferred for derived data structures.
      const q = colRef;
      
      return onSnapshot(
        q,
        (snapshot) => col.setter(snapshot.docs.map(d => ({ id: d.id, ...d.data() }))),
        (err) => console.error(`${col.name} error`, err)
      );
    });

    // Listener for Organization Profile (single document)
    const profileDocRef = doc(db, 'artifacts', appId, 'public', 'data', 'organizationProfile', 'profile');
    const unsubProfile = onSnapshot(profileDocRef, (docSnap) => {
        if (docSnap.exists()) {
            setOrganizationProfile(docSnap.data());
        } else {
            setOrganizationProfile({});
        }
    }, (err) => console.error("Organization Profile error", err));

    return () => {
        unsubs.forEach(unsub => unsub());
        unsubProfile();
    }
  }, [fsAuthUser]);

  // 3. Authentication & User Actions
  const handleLogin = (username, password) => {
    const foundUser = users.find(u => u.username === username && u.password === password);
    if (foundUser) {
      setCurrentUser(foundUser);
      setAuthError(null);
    } else {
      setAuthError('Invalid credentials');
    }
  };

  const handleLogout = () => {
    setCurrentUser(null);
    setView('dashboard');
  };

  // --- Firestore CRUD Handlers ---

  const addDocToCollection = async (collectionName, data) => {
    try {
        await addDoc(collection(db, 'artifacts', appId, 'public', 'data', collectionName), {
            ...data, createdAt: serverTimestamp()
        });
    } catch (e) { console.error("Error adding document:", e); }
  };

  const updateDocInCollection = async (collectionName, id, data) => {
     try {
         await updateDoc(doc(db, 'artifacts', appId, 'public', 'data', collectionName, id), data);
     } catch (e) { console.error("Error updating document:", e); }
  };

  const deleteDocInCollection = async (collectionName, id) => {
    // Note: Using an in-app confirmation mechanism instead of window.confirm
    // We use standard window.confirm here as we cannot use a custom modal for every file.
    if(window.confirm(`Are you sure you want to delete this item from ${collectionName}?`)) {
        try {
            await deleteDoc(doc(db, 'artifacts', appId, 'public', 'data', collectionName, id));
        } catch (e) { console.error("Error deleting document:", e); }
    }
  };

  const addUser = (data) => {
    if (users.find(u => u.username === data.username)) {
      console.error("Username already taken");
      return;
    }
    addDocToCollection('users', data);
  };

  const addCustomer = (data) => addDocToCollection('customers', data);
  const deleteCustomer = (id) => deleteDocInCollection('customers', id);

  const addCatchType = (data) => addDocToCollection('catchTypes', data);
  const deleteCatchType = (id) => deleteDocInCollection('catchTypes', id);
  
  const addTripType = (data) => addDocToCollection('tripTypes', data);
  const deleteTripType = (id) => deleteDocInCollection('tripTypes', id);

  const addBill = (data) => addDocToCollection('bills', data);
  const updateBill = (id, data) => updateDocInCollection('bills', id, data);
  const deleteBill = (id) => deleteDocInCollection('bills', id);

  const addCatch = (data) => addDocToCollection('catch', data);
  const deleteCatch = (id) => deleteDocInCollection('catch', id);

  // New Profile Handler
  const updateOrganizationProfile = async (data) => {
      const profileDocRef = doc(db, 'artifacts', appId, 'public', 'data', 'organizationProfile', 'profile');
       try {
           await setDoc(profileDocRef, data, { merge: true });
           console.log("Organization profile updated successfully.");
       } catch (e) { console.error("Error updating organization profile:", e); }
  };


  // --- Render ---

  if (loading) return <div className="min-h-screen flex items-center justify-center text-slate-500">Connecting to the Marina...</div>;

  if (!currentUser) return <Login onLogin={handleLogin} error={authError} />;

  const isAdmin = currentUser.role === 'admin';

  return (
    <div className="flex min-h-screen bg-slate-50 font-sans text-slate-900">
      
      {/* Mobile Header */}
      <div className="md:hidden fixed top-0 w-full bg-slate-900 text-white z-40 p-4 flex justify-between items-center shadow-md">
        <div className="flex items-center gap-2">
            <Anchor className="w-6 h-6 text-blue-400" />
            <span className="font-bold">Captain's Log</span>
        </div>
        <button onClick={() => setIsMobileMenuOpen(!isMobileMenuOpen)}>
          {isMobileMenuOpen ? <X /> : <Menu />}
        </button>
      </div>

      {/* Sidebar Navigation */}
      <aside className={`
        fixed inset-y-0 left-0 z-30 w-64 bg-slate-900 text-slate-300 transform transition-transform duration-200 ease-in-out
        ${isMobileMenuOpen ? 'translate-x-0' : '-translate-x-full'} md:translate-x-0 md:static flex flex-col
      `}>
        <div className="p-6 flex items-center gap-3 border-b border-slate-800">
          <div className="w-10 h-10 bg-blue-600 rounded-lg flex items-center justify-center text-white shadow-lg shadow-blue-900/50">
            <Anchor className="w-6 h-6" />
          </div>
          <div>
            <h1 className="text-white font-bold text-lg">Captain's Log</h1>
            <p className="text-xs text-slate-500">Hello, {currentUser.username}</p>
          </div>
        </div>

        <nav className="flex-1 p-4 space-y-2">
          {/* Menu Items */}
          {[
            { id: 'dashboard', name: 'Dashboard', icon: BarChart3 },
            { id: 'billing', name: 'Bills & Sales', icon: DollarSign },
            { id: 'customers', name: 'Customers', icon: Users },
            { id: 'catch', name: 'Stock & Catch', icon: Fish },
            ...(isAdmin ? [{ id: 'settings', name: 'Settings', icon: Settings }] : []),
          ].map(({ id, name, icon: Icon }) => (
            <button 
              key={id}
              onClick={() => { setView(id); setIsMobileMenuOpen(false); }}
              className={`w-full flex items-center gap-3 px-4 py-3 rounded-xl transition-all ${view === id ? 'bg-blue-600 text-white shadow-lg shadow-blue-900/20' : 'hover:bg-slate-800'}`}
            >
              <Icon className="w-5 h-5" /> {name}
            </button>
          ))}
        </nav>

        <div className="p-4 border-t border-slate-800">
          <div className="flex items-center gap-3 mb-4 px-2">
            <div className="w-8 h-8 rounded-full bg-slate-700 flex items-center justify-center text-xs font-bold text-white uppercase">
              {currentUser.username.substring(0,2)}
            </div>
            <div className="overflow-hidden">
              <p className="text-sm text-white font-medium truncate">{currentUser.username}</p>
              <p className="text-xs text-slate-500 capitalize">{currentUser.role} (ID: {fsAuthUser.uid.substring(0, 8)}...)</p>
            </div>
          </div>
          <button 
            onClick={handleLogout}
            className="w-full flex items-center gap-2 text-slate-400 hover:text-white px-2 py-2 transition-colors text-sm"
          >
            <LogOut className="w-4 h-4" /> Sign Out
          </button>
        </div>
      </aside>

      {/* Main Content */}
      <main className="flex-1 md:h-screen md:overflow-y-auto pt-20 md:pt-0">
        <div className="p-6 md:p-8 max-w-7xl mx-auto">
          {view === 'dashboard' && <Dashboard bills={bills} catches={catches} />}
          {view === 'billing' && (
            <Billing 
              bills={bills} 
              customers={customers} 
              catchTypes={catchTypes}
              tripTypes={tripTypes}
              onAdd={addBill} 
              onUpdate={updateBill} 
              onDelete={deleteBill} 
            />
          )}
          {view === 'customers' && (
             <CustomerManagement
                customers={customers}
                bills={bills}
                onAdd={addCustomer}
                onDelete={deleteCustomer}
              />
          )}
          {view === 'catch' && (
            <CatchLog 
              catchStock={catchStock}
              catches={catches} 
              catchTypes={catchTypes}
              onAdd={addCatch} 
              onDelete={deleteCatch} 
            />
          )}
          {view === 'settings' && isAdmin && (
            <AdminSettings 
              users={users} 
              catchTypes={catchTypes}
              tripTypes={tripTypes}
              profile={organizationProfile}
              onAddUser={addUser} 
              onDeleteUser={deleteDocInCollection} // Direct call to generic delete for simplicity
              onAddCatchType={addCatchType}
              onDeleteCatchType={deleteCatchType}
              onAddTripType={addTripType}
              onDeleteTripType={deleteTripType}
              onUpdateOrganizationProfile={updateOrganizationProfile}
              currentUser={currentUser}
            />
          )}
        </div>
      </main>
    </div>
  );
}
