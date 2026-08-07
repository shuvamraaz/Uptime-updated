<template>
    <teleport to="body">
        <div
            v-if="content"
            ref="tooltip"
            class="tooltip-wrapper"
            :style="tooltipStyle"
            :class="{ 'tooltip-above': position === 'above' }"
        >
            <div class="tooltip-content">
                <slot :content="content">
                    <div class="tooltip-header">
                        <font-awesome-icon :icon="icon" class="status-icon" :class="statusClass" />
                        <span class="tooltip-status" :class="statusClass">{{ statusText }}</span>
                    </div>
                    <div class="tooltip-divider"></div>
                    <div class="tooltip-time">{{ timeText }}</div>
                    <div v-if="content?.msg" class="tooltip-message">{{ content.msg }}</div>
                </slot>
            </div>
            <div class="tooltip-arrow" :class="{ 'arrow-above': position === 'above' }"></div>
        </div>
    </teleport>
</template>

<script>
import { DOWN, UP, PENDING, MAINTENANCE } from "../util.ts";

export default {
    name: "Tooltip",
    props: {
        /** Whether tooltip is visible */
        visible: {
            type: Boolean,
            default: false,
        },
        /** Content object to display */
        content: {
            type: Object,
            default: null,
        },
        /** X position (viewport coordinates) */
        x: {
            type: Number,
            default: 0,
        },
        /** Y position (viewport coordinates) */
        y: {
            type: Number,
            default: 0,
        },
        /** Position relative to target element */
        position: {
            type: String,
            default: "below",
            validator: (value) => ["above", "below"].includes(value),
        },
    },
    computed: {
        tooltipStyle() {
            return {
                left: this.x + "px",
                top: this.y + "px",
            };
        },

        statusText() {
            if (!this.content || this.content === 0) {
                return this.$t("Unknown");
            }

            switch (this.content.status) {
                case DOWN:
                    return this.$t("Down");
                case UP:
                    return this.$t("Up");
                case PENDING:
                    return this.$t("Pending");
                case MAINTENANCE:
                    return this.$t("Maintenance");
                default:
                    return this.$t("Unknown");
            }
        },

        statusClass() {
            if (!this.content || this.content === 0) {
                return "status-empty";
            }

            switch (this.content.status) {
                case DOWN:
                    return "status-down";
                case UP:
                    return "status-up";
                case PENDING:
                    return "status-pending";
                case MAINTENANCE:
                    return "status-maintenance";
                default:
                    return "status-unknown";
            }
        },

        timeText() {
            if (!this.content || this.content === 0) {
                return "";
            }
            return this.$root.datetime(this.content.time);
        },

        icon() {
            if (!this.content || this.content === 0) {
                return ['fas', 'question-circle'];
            }
            switch (this.content.status) {
                case DOWN: return ['fas', 'times-circle'];
                case UP: return ['fas', 'check-circle'];
                case PENDING: return ['fas', 'hourglass-half'];
                case MAINTENANCE: return ['fas', 'wrench'];
                default: return ['fas', 'question-circle'];
            }
        },
    },
};
</script>

<style lang="scss" scoped>
@import "../assets/vars.scss";

.tooltip-wrapper {
    position: fixed;
    z-index: 9999;
    pointer-events: none;
    transform: translateX(-50%);

    .tooltip-content {
        background: #1f2328;
        border: 1px solid rgba(255, 255, 255, 0.1);
        border-radius: 8px;
        padding: 12px 16px;
        box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1), 0 2px 4px -1px rgba(0, 0, 0, 0.06);
        min-width: 180px;
        text-align: left;
        position: relative;

        .tooltip-header {
            display: flex;
            align-items: center;
            margin-bottom: 12px;
            
            .status-icon {
                margin-right: 8px;
                font-size: 16px;
                
                &.status-up { color: $primary; }
                &.status-down { color: $danger; }
                &.status-pending { color: $warning; }
                &.status-maintenance { color: $maintenance; }
                &.status-empty { color: $secondary-text; }
            }
            
            .tooltip-status {
                font-size: 15px;
                font-weight: 600;
                color: #ffffff;
                text-transform: capitalize;
            }
        }

        .tooltip-divider {
            height: 1px;
            background: rgba(255, 255, 255, 0.1);
            margin-bottom: 8px;
        }

        .tooltip-time {
            color: #9ca3af;
            font-size: 13px;
        }

        .tooltip-message {
            color: #f3f4f6;
            font-size: 12px;
            margin-top: 6px;
            padding-top: 6px;
            border-top: 1px solid rgba(255, 255, 255, 0.1);
        }
    }

    .tooltip-arrow {
        position: absolute;
        left: 50%;
        transform: translateX(-50%);
        width: 12px;
        height: 6px;
        overflow: hidden;
        top: -6px;

        &::before {
            content: "";
            position: absolute;
            left: 50%;
            top: 100%;
            transform: translateX(-50%) translateY(-50%) rotate(45deg);
            width: 8px;
            height: 8px;
            background: rgba(17, 24, 39, 0.95);
            border: 1px solid rgba(75, 85, 99, 0.3);
            border-bottom: none;
            border-right: none;
        }

        &.arrow-above {
            top: auto;
            bottom: -6px;

            &::before {
                top: 0%;
                transform: translateX(-50%) translateY(-50%) rotate(225deg);
                border: 1px solid rgba(75, 85, 99, 0.3);
                border-bottom: none;
                border-right: none;
            }
        }
    }

    // Smooth entrance animation
    animation: tooltip-fade-in 0.2s $easing-out;

    &.tooltip-above {
        transform: translateX(-50%) translateY(-8px);

        .tooltip-content::before {
            top: auto;
            bottom: -1px;
        }
    }
}

// Dark theme adjustments
.dark .tooltip-wrapper {
    .tooltip-content {
        background: #1f2328;
        border-color: rgba(255, 255, 255, 0.1);

        .tooltip-status {
            color: #ffffff;
        }
    }

    .tooltip-arrow {
        display: none; // Hiding arrow as per Better Stack design
    }
}

@keyframes tooltip-fade-in {
    from {
        opacity: 0;
        transform: translateX(-50%) translateY(4px);
    }

    to {
        opacity: 1;
        transform: translateX(-50%) translateY(0);
    }
}

// Accessibility improvements

@media (prefers-reduced-motion: reduce) {
    .tooltip-wrapper {
        animation: none !important;
    }
}
</style>
