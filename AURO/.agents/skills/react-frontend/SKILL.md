---
name: react-frontend
description: React 18/19, TypeScript, Vite, and Tailwind CSS design guidelines for AURO academic management dashboard and timetable matrix.
---

# React Frontend Skill for AURO

## Guidelines
1. **Academic Matrix Layout**: Render timetables as a 2D multi-axis grid with Days on the X-axis and Time Intervals on the Y-axis.
2. **Visual Hierarchy & State Colors**:
   - Lectures: Sky Blue (`bg-sky-50 border-sky-200 text-sky-900`)
   - Laboratories: Purple (`bg-purple-50 border-purple-200 text-purple-900`)
   - Proposed Recoveries: Amber with dashed border (`bg-amber-50 border-amber-300 text-amber-900 border-dashed`)
   - Conflicted Slots: High-contrast Rose (`bg-rose-50 border-rose-300 text-rose-900 ring-1 ring-rose-400`)
3. **Responsive Primitives**: Ensure horizontal scrolling on smaller viewports (`min-w-[960px]`) so matrices never deform.
4. **State Management**: Keep filter states (Batch, Faculty, Room) localized and fast without full-page reloads.

