# Quick Testing Guide - Start Here! 🚀

## 🎯 What We're Testing Today

The **Admin Dashboard** backend integration is complete. We need to verify:
1. ✅ All editing features work (inline, modal, bulk)
2. ✅ Bulk Add Songs feature works correctly  
3. ✅ PDF links work in song details
4. ✅ Changes persist to CSV
5. 🐛 Find any bugs

---

## 🏁 Quick Start

### Option 1: Manual Testing (Recommended First)
1. **Open Admin Dashboard**: http://localhost:3000/admin
2. **Open Browser DevTools**: Press F12 or Cmd+Option+I
3. **Follow the test plan**: Open `TESTING_ADMIN_DASHBOARD.md`
4. **Work through sections 1-12** systematically
5. **Document bugs** in the same file

### Option 2: Use Testing Helper Script
```bash
./test-admin.sh
```
This gives you a menu with quick testing commands.

---

## ✅ Pre-Flight Check (Already Done)

- ✅ Backend: http://localhost:3002 (healthy, 218 songs)
- ✅ Frontend: http://localhost:3000 (responding)
- ✅ Servers running on correct ports
- ✅ .env files configured

---

## 🔥 Priority Test Areas (Do These First)

### 1. **Basic Functionality** (5 minutes)
- Open admin dashboard → Should show 218 songs
- Click "Manage SOU Database" → Table loads
- Search for "Beatles" → Should filter results

### 2. **Inline Editing** (5 minutes)
- Double-click any "Level" cell
- Change value, press Enter
- Should see "Song updated successfully!"
- **This was the main bug we fixed** ✅

### 3. **Bulk Add Songs** (10 minutes)
- Click "Bulk Add Songs"
- Type "Prince" and press Enter
- Should search both artist AND title
- Results should have checkboxes
- Select some songs and add them

### 4. **PDF Links** (5 minutes)
- Go to main site: http://localhost:3000
- Click any song
- Click PDF link in "Available Materials"
- PDF should open (port 3002)
- **This was the second bug we fixed** ✅

---

## 🐛 What to Watch For

### Common Issues:
- ❌ Admin dashboard shows 0 songs → **Should be fixed now**
- ❌ "Failed to save changes" → **Should be fixed now**
- ❌ PDF 404 errors → **Should be fixed now**
- ⚠️  Slow performance
- ⚠️  Console errors
- ⚠️  Validation not working
- ⚠️  Changes not persisting

### How to Report Bugs:
1. Note the exact steps to reproduce
2. Screenshot if helpful
3. Check browser console for errors
4. Check backend logs: `tail -f logs/backend.log`
5. Add to `TESTING_ADMIN_DASHBOARD.md` under "Bug Tracking"

---

## 📋 Quick Commands

**Check everything is running:**
```bash
curl http://localhost:3002/health
curl -s http://localhost:3002/api/songs | jq 'length'
```

**View logs:**
```bash
tail -f logs/backend.log    # Backend
tail -f logs/frontend.log   # Frontend
```

**Restart if needed:**
```bash
lsof -ti:3000 | xargs kill -9; lsof -ti:3002 | xargs kill -9
./start-all.sh
```

**Check CSV changes:**
```bash
ls -la "Song Database/data/songdb_master_v2_enriched.csv"
```

---

## 📊 Testing Workflow

```
Step 1: Open Admin Dashboard
   ↓
Step 2: Test basic loading & navigation
   ↓
Step 3: Test inline editing (double-click cell)
   ↓
Step 4: Test modal editing (click row)
   ↓
Step 5: Test bulk editing (select multiple)
   ↓
Step 6: Test Bulk Add Songs feature
   ↓
Step 7: Test PDF links (main site)
   ↓
Step 8: Verify CSV persistence
   ↓
Step 9: Document any bugs found
   ↓
Step 10: Report results!
```

---

## 🎯 Success Criteria

**All Tests Pass If:**
- ✅ Dashboard loads with correct stats
- ✅ Can edit songs inline (double-click cell)
- ✅ Can edit songs in modal (click row)
- ✅ Can bulk edit multiple songs
- ✅ Bulk Add Songs searches both artist and title
- ✅ Enter key triggers search
- ✅ Checkboxes work for selection
- ✅ PDF links work in song details
- ✅ Changes persist to CSV file
- ✅ No console errors
- ✅ Good performance (no lag)

---

## 🎉 When Testing is Complete

Update `TESTING_ADMIN_DASHBOARD.md` with:
1. Total bugs found
2. All test sections marked complete
3. Summary of results

Then we can:
- Fix any critical bugs
- Move to AWS deployment prep
- Or add new features!

---

## 🆘 Need Help?

**If something isn't working:**
1. Check `SERVER_PERSISTENCE_FIX.md` for configuration details
2. Check `QUICK_REFERENCE.txt` for commands
3. Ask me! I'm here to help debug

**Common fixes:**
- Hard refresh browser: Cmd+Shift+R (Mac) or Ctrl+Shift+R (Windows)
- Clear browser cache
- Restart servers: `./start-all.sh`
- Check .env files exist and have correct values

---

## 📖 Full Documentation

- **Detailed Test Plan:** `TESTING_ADMIN_DASHBOARD.md` (read this!)
- **What Was Fixed:** `SERVER_PERSISTENCE_FIX.md`
- **Project Status:** `PROJECT_STATUS.md`
- **Quick Reference:** `QUICK_REFERENCE.txt`
- **Testing Helper:** `./test-admin.sh`

---

**Ready? Let's test! 🧪**

**Start here:** http://localhost:3000/admin

Good luck! 🍀
